<script> 
  import App from "../fb";
  import { AuthCredential, getAuth } from "firebase/auth";
  import { addDoc, collection, getFirestore, onSnapshot } from "firebase/firestore";
  import { DataSnapshot, query } from "firebase/database";
  import { onMount } from "svelte";

  const auth = getAuth();
  const db = getFirestore(App);
  let uid;
  let lat, lon;
  let trips = [];

  onMount(() => {
    typeof localStorage !== 'undefined';
    uid = localStorage.getItem("uid") || auth.currentUser.uid;
    console.log(uid);
    readLongLat();
    readData();
    
  })

  function signOut() {
    console.log("YOU SMEL");
    auth.signOut().then(() => {
          localStorage.removeItem("uid");
          window.location.href = '/';
    });
  }

  function readLongLat() {
    const q = query(collection(db, `users/${uid}/info`));
    onSnapshot(q, (querySnapshot) => {
      querySnapshot.forEach((doc) => {
        let obj = {...doc.data() };
        lat = obj.latitude;
        lon = obj.longitude;
        console.log(lat);
        console.log(lon);
      });
    });
  }

  let data = [];
  function readData() {
    const q = query(collection(db, "coords"));
    onSnapshot(q, (querySnapshot) => {
      querySnapshot.forEach((doc) => {
        let obj = {...doc.data() };
        const q2 = query(collection(db, `users/${obj.uid}/info`));
        onSnapshot(q2, (querySnapshot2) => {
          querySnapshot2.forEach((doc2) => {
            obj = {...obj, ...doc2.data() };
            obj.distance = distance(obj.latitude, obj.longitude);
            obj.trips = [];
            const q3 = query(collection(db, `trips`));
            onSnapshot(q3, (querySnapshot3) => {
              querySnapshot3.forEach((doc3) => {
                let tempObj = { ...doc3.data() };
                if (tempObj.uid === obj.uid) {
                  // Check if tempObj is in obj.trips
                  let found = false;
                  for (let i = 0; i < obj.trips.length; i++) {
                    if (obj.trips[i].uid === tempObj.uid && obj.trips[i].location === tempObj.location && obj.trips[i].date === tempObj.date) {
                      found = true;
                      break;
                    }
                  }
                  if (!found)
                    obj.trips.push(tempObj);
                  console.log(obj.trips)
                }
                sortData();
              });
            });
            data.push(obj);
          });
        });
      });
    });
  }

  function sortData() {
    data.sort((a, b) => (a.distance > b.distance) ? 1 : -1);
    data = data.filter((item) => item.uid !== uid);
    trips = [];
    for (let i = 0; i < data.length; i++) {
      for (let j = 0; j < data[i].trips.length; j++) {
        let obj = { ...data[i].trips[j], index:i };
        let found = false;
        for (let i = 0; i < trips.length; i++) {
          if (trips[i].uid === obj.uid && trips[i].location === obj.location && trips[i].date === obj.date) {
            found = true;
            break;
          }
        }
        if (!found)
          trips.push(obj);
      }
    }

  }

  // Functions that finds the distance in miles between two latitude and longitude coordinates
  function distance(lat2, lon2) {
    var p = 0.017453292519943295;    // Math.PI / 180
    var c = Math.cos;
    var a = 0.5 - c((lat2 - lat) * p)/2 + 
            c(lat * p) * c(lat2 * p) * 
            (1 - c((lon2 - lon) * p))/2;

    return 7918 * Math.asin(Math.sqrt(a)); // 2 * R; R = 6371 km
  }
  

  let addARide = true;
  function addRide() {
    if (addARide) {
      addARide = false;
      const submitButton = document.querySelector("button");
      const locationInput = document.querySelector("#where");
      const timeInput = document.querySelector("#time");
      submitButton.setAttribute("disabled", "");
      locationInput.setAttribute("disabled", "");
      timeInput.setAttribute("disabled", "");
      // Get where do you go value
      let where = document.getElementById("where").value;
      let time = document.getElementById("time").value;
      addDoc(collection(db, `trips/`), {
              location: where,
              time: time,
              uid: uid
          }
      ).then((docRef) => {
          console.log("Document written with ID: ", docRef.id);
          document.getElementById("where").value = '';
          time = document.getElementById("time").value = '';
          submitButton.removeAttribute("disabled")
          locationInput.removeAttribute("disabled")
          timeInput.removeAttribute("disabled")
          addARide = true;
      });
    }
  }
</script>


<!--Navbar for top of dashboard-->
<nav class="navbar bg-light" >
  <div class="container-fluid">
    <h1 class="navbar-header">Find A Carpool</h1>
    <ul class="nav navbar-nav">
      <button class="btn btn-danger navbar-btn" style="background-color:#122861; border:#122681" on:click={signOut}>Sign Out</button>
    </ul>
  </div>
</nav>
<div class="image">
  <img src="https://media.discordapp.net/attachments/1016541378962014240/1023569981323100220/unknown.png" alt="">

</div>

<div class="grid-container">
    

      <div class="charts">

        <div class="charts-card" id="addRideForm">
          <p class="chart-title">Add a Ride</p>
          <form on:submit|preventDefault={addRide} class="row g-3">
            <div class="col-12">
              <input type="text" id="where" class="form-control" placeholder="Where do you go?" required>
            </div>
            <div class="col-12">
              <input type="time" id="time" class="form-control" placeholder="Departure Time" required>
            </div>
            
            
            <button type="submit" class="btn btn-primary">Submit</button>
          </form>
          
          <div id="bar-chart"></div>
        </div>

        <div class="charts-card" id="potentialCarpoolers">
          <p class="chart-title">Potential Carpoolers</p> 
          <table class="contentTable">
            <tr>
              <th>Distance (mi)</th>
              <th>Name</th>
              <th>Departure Time</th>
              <th>Destination</th>
            </tr>
            {#each trips as {location, time, index}}
              {#if location}
              <tr>
                <td align="center">{data[index].distance.toFixed(2)}</td>
                <td align="center"><a href={`mailto:${data[index].email}`}>{data[index].firstName}</a></td>
                <td align="center">{time}</td>
                <td align="center">{location}</td>
              </tr>
              {/if}
            {/each}
          </table>
          <div id="area-chart"></div>
        </div>

      </div>

  </div>

<style>
  :global(body) {
    background:-webkit-linear-gradient(right, #112d5f, #534ec4, #3885e5);
  }
  
  tr th{
    border: 1px solid blue;
    padding: 10px;
    text-align: center;
  }

  input:hover{
    background-color:#d8d7ed;
  }

  .image {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 2rem;
  }

  .grid-container{
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    margin-top: 2rem;
  }
  nav {
    background-color: blue;
  }

  .navbar-header {
    font-weight: bold;
    font-size: 500%;
    background: -webkit-linear-gradient(right, #112d5f, #534ec4, #3885e5);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;


  }

.grid-container {
  display: grid;
  grid-template-columns: auto auto auto;
  grid-template-rows: 0.2fr 3fr;
  grid-template-areas: "auto auto auto";
  height: 100vh;
}

p{
  color: #112d5f;
}

.charts {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.charts-card {
  background-color: #ffffff;
  margin-bottom: 20px;
  padding: 25px;
  box-sizing: border-box;
  -webkit-column-break-inside: avoid;
  border: 1px solid #d2d2d3;
  border-radius: 5px;
  box-shadow: 0 6px 7px -4px rgba(0, 0, 0, 0.2);
}

.chart-title {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 22px;
  font-weight: 600;
}

::-webkit-scrollbar {
  width: 5px;
  height: 6px;
}

::-webkit-scrollbar-track {
  box-shadow: inset 0 0 5px #a5aaad;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb {
  background-color: #4f35a1;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb:hover {
  background-color: #a5aaad;
}

@media screen and (max-width: 768px) {
  .charts {
    grid-template-columns: 1fr;
    margin-top: 30px;
  }
}
</style>
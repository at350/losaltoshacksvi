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
    // typeof localStorage !== 'undefined';
    readLongLat();
    readData();
    // sortData();
    // console.log(data)
    // Remove any items in data that have the same uid as the current user
    // uid =  auth.currentUser.uid;
    // loop through data
  })

  function signOut() {
    console.log("YOU SMEL");
    auth.signOut().then(() => {
          localStorage.removeItem("uid");
          window.location.href = '/';
    });
  }

  function readLongLat() {
    // uid = auth.currentUser.uid || localStorage.getItem("uid");
    // console.log(uid);
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
                if (tempObj.uid === obj.uid)
                  obj.trips.push(tempObj);
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
    console.log(data)
    // Move all the trips from data into trips
    for (let i = 0; i < data.length; i++) {
      console.log(data[i].trips)
      for (let j = 0; j < data[i].trips.length; j++) {
        console.log(data[i].trips.length)
        let obj = { ...data[i].trips[j], index:i };
        console.log(obj);
        trips.push(obj);
      }
    }

    console.log(trips)

    // console.log("HI")
    // console.log(data)
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
  /*
  <li class="dropdown">
        <a class="dropdown-toggle" data-toggle="dropdown" href="Youtube.com">Help
        <span class="caret"></span></a>
        <ul class="dropdown-menu">
          <li><a href="Youtube.com">Help</a></li>
          <li><a href="Youtube.com">Help2</a></li>
        </ul>
      </li>
      */

  function addRide() {
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
    });
  }
</script>


<!--Navbar for top of dashboard-->
<nav class="navbar bg-light" >
  <div class="container-fluid">
    <h1 class="navbar-header">Find A Carpool</h1>
    <ul class="nav navbar-nav">
      <button class="btn btn-danger navbar-btn" style="background-color:blue; border:blue" on:click={signOut}>Sign Out</button>
    </ul>
  </div>
</nav>

<div class="grid-container">
    <!--<h1 class="font-weight-bold" style="color:#122861; font-size:40px;"><pre><br> DASHBOARD</pre></h1>-->

      <!-- Weird stuff ahhhhh -->
      <!-- <py-script output="mpl">
          from helper import *
          import tkinter as tk
          import matplotlib.pyplot as plt
          import numpy as np
          from matplotlib import colors
          from matplotlib.ticker import PercentFormatter
          import matplotlib.mlab as mlab


          distances = [23.4, 0.45, 34, 0.45, 0.78, 4.5345, 6.32333, 2.0, 3] 

          num_bins = 5
          n, bins, patches = plt.hist(distances, num_bins, facecolor='blue', alpha=0.5)
          plt.xlabel('Distance from you (miles)')
          plt.ylabel('Number of Potential Carpoolers')
          plt.title(r'Carpoolers Near You')
          plt.savefig('carpoolers.png', bbox_inches='tight', transparent = True)
          plt.show()
      </py-script>   -->
      <!-- <div class="main-cards">

       <div class="card" onclick="location.href='/home?/requestRides'">  This was meant to be clickable and it'd take you to a new page but we kinda scrapped that idea
          <div class="card-inner">
            <p class="text-primary">Request a Ride</p>
            <span class="material-icons-outlined text-blue">inventory_2</span>
          </div>
          <span class="text-primary font-weight-bold">(This is where ride requests occur)</span>
        </div>

        <div class="card">
          <div class="card-inner">
            <p class="text-primary">Offer a Ride</p>
            <span class="material-icons-outlined text-orange">add_shopping_cart</span>
          </div>
          <span class="text-primary font-weight-bold">(This is where you pick people up)</span>
        </div>

        <div class="card">
          <div class="card-inner">
            <p class="text-primary">Update Info</p>
            <span class="material-icons-outlined text-green">shopping_cart</span>
          </div>
          <span class="text-primary font-weight-bold">(Update address and pickup time)</span>
        </div>

        <div class="card">
          <div class="card-inner">
            <p class="text-primary">Ride History</p>
            <span class="material-icons-outlined text-red">notification_important</span>
          </div>
          <span class="text-primary font-weight-bold">(For the aesthetic, not to be implemented)</span>
        </div>

      </div> -->

      <div class="charts">

        <div class="charts-card">
          <p class="chart-title">Add a Ride</p>
          <form on:submit|preventDefault={addRide}>
            <input type="text" id="where" placeholder="Where do you go?">
            <input type="time" id="time" placeholder="Departure Time">
            
            <!-- <p>Are you someone driving or are you a person looking for a carpool?</p>
            <input type="radio" id="driving" name="a" value="0">
            <label for="driving">I am driving</label><br>
            <input type="radio" id="carpool" name="a" value="1">
            <label for="carpool">I am looking for a carpool</label><br> -->
            
            <button type="submit" class="haha" style="margin:10px, padding:5px">Submit</button>
          </form>
          
          <div id="bar-chart"></div>
        </div>

        <div class="charts-card">
          <p class="chart-title">Potential Carpoolers</p> 
          <table class="contentTable">
            <tr>
              <th>Distance</th>
              <th>Name</th>
              <th>Departure Time</th>
              <th>Destination</th>
            </tr>
            {#each trips as {location, time, index}}
              {#if location}
              <tr>
                <td>{data[index].distance.toFixed(2)}</td>
                <td><a href={`mailto:${data[index].email}`}>{data[index].firstName}</a></td>
                <td>{time}</td>
                <td>{location}</td>
              </tr>
              {/if}
            {/each}
          </table>
          <div id="area-chart"></div>
        </div>

      </div>

  </div>

<style>

  table .contentTable {
    border:black;
    padding: 5px;;
  }
  .contentTable th{b}

  input:hover{
    background-color:#d8d7ed;
  }
  .haha:hover{
    background-color:#d8d7ed;
  }

  .grid-container{background:-webkit-linear-gradient(right, #112d5f, #534ec4, #3885e5);}
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
  input{
    margin: 5px;
  }
   body {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  background-color: #e6e8ed;
  color: #666666;
  font-family: "Montserrat", sans-serif;
}

.material-icons-outlined {
  vertical-align: middle;
  line-height: 1px;
}

/*.text-primary {
  color: #666666;
}

.text-blue {
  color: #246dec;
}

.text-red {
  color: #cc3c43;
}

.text-green {
  color: #367952;
}

.text-orange {
  color: #f5b74f;
}*/

.font-weight-bold {
  font-weight: 600;
}

.grid-container {
  display: grid;
  grid-template-columns: auto auto auto;
  grid-template-rows: 0.2fr 3fr;
  grid-template-areas: "auto auto auto";
  height: 100vh;
}


/* ---------- HEADER ---------- */

.header {
  grid-area: header;
  height: 200px;
  background-color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 30px 0 30px;
  box-shadow: 0 6px 7px -4px rgba(0, 0, 0, 0.2);
}

.menu-icon {
  display: none;
}
p{
  color: red;
}

/* ---------- SIDEBAR ---------- */

#sidebar {
  grid-area: sidebar;
  height: 100%;
  background-color: #21232d;
  color: #9799ab;
  overflow-y: auto;
  transition: all 0.5s;
  -webkit-transition: all 0.5s;
}

.sidebar-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 20px 20px 20px;
  margin-bottom: 30px;
}

.sidebar-title > span {
  display: none;
}

.sidebar-brand {
  margin-top: 15px;
  font-size: 20px;
  font-weight: 700;
}

.sidebar-list {
  padding: 0;
  margin-top: 15px;
  list-style-type: none;
}

.sidebar-list-item {
  padding: 20px 20px 20px 20px;
}

.sidebar-list-item:hover {
  background-color: rgba(255, 255, 255, 0.2);
  cursor: pointer;
}

.sidebar-responsive {
  display: inline !important;
  position: absolute;
}

/* ---------- MAIN ---------- */

.main-container {
  grid-area: main;
  overflow-y: auto;
  padding: 20px 20px;
}

.main-title {
  display: flex;
  justify-content: space-between;
}

.main-title > p {
  font-size: 20px;
}

/*.main-cards {
  display: grid;
  grid-template-columns: auto auto auto;
  gap: 20px;
  margin: 20px 0;
}

.card {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  padding: 25px;
  background-color: #ffffff;
  box-sizing: border-box;
  border: 1px solid #d2d2d3;
  border-radius: 5px;
  box-shadow: 0 6px 7px -4px rgba(0, 0, 0, 0.2);
}

.card:first-child {
  border-left: 7px solid #246dec;
}

.card:nth-child(2) {
  border-left: 7px solid #f5b74f;
}

.card:nth-child(3) {
  border-left: 7px solid #367952;
}

.card:nth-child(4) {
  border-left: 7px solid #cc3c43;
}

.card > span {
  font-size: 20px;
  font-weight: 600;
}

.card-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.card-inner > p {
  font-size: 18px;
}

.card-inner > span {
  font-size: 35px;
}
*/
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


/* ---------- SCROLLBARS ---------- */

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


/* ---------- MEDIA QUERIES ---------- */


/* Medium <= 992px */
@media screen and (max-width: 992px) {
  

  #sidebar {
    display: none;
  }

  .menu-icon {
    display: inline;
  }

  .sidebar-title > span {
    display: inline;
  }
}

/* Small <= 768px */
@media screen and (max-width: 768px) {
  .main-cards {
    grid-template-columns: 1fr;
    gap: 10px;
    margin-bottom: 0;
  }

  .charts {
    grid-template-columns: 1fr;
    margin-top: 30px;
  }
}

/* Extra Small <= 576px */
@media screen and (max-width: 576px) {
  .header-left {
    display: none;
  }
}
</style>
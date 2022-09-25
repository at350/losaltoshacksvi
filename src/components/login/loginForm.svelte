<script>
    import App from "../../routes/fb";
    import { goto } from "$app/navigation";
    import { getAuth, signInWithEmailAndPassword, createUserWithEmailAndPassword } from "firebase/auth";
    import { addDoc, collection, getFirestore } from "firebase/firestore";
    import axios from "axios";
    export let type;

    const auth = getAuth();

    let showNotFound = false;

    const db = getFirestore(App);
    let uid;

    function login() {
        let email = document.getElementById("email").value;
        let password = document.getElementById("password").value;
        if (type == "Login") {
            signInWithEmailAndPassword(auth, email, password)
            .then((userCredential) => {
                const user = userCredential.user;
                localStorage.setItem("uid", user.uid)
                window.location.href='/home';
            })
            .catch((error) => {
                console.error(error);
                showNotFound = true;
            });
        }
        else {
            createUserWithEmailAndPassword(auth, email, password)
            .then((userCredential) => {
                const user = userCredential.user;
                localStorage.setItem("uid", user.uid);
                uid = user.uid;
                writeUserDetails(email);
            })
            .catch((error) => {
                console.error(error);
            });
        }
    }

    function writeUserDetails(email) {
      let firstName = document.getElementById("firstName").value;
      let lastName = document.getElementById("lastName").value;
      let address = document.getElementById("inputAddress").value;
      let address2 = document.getElementById("inputAddress2").value;
      let inputCity = document.getElementById("inputCity").value;
      let inputState = document.getElementById("inputState").value;
      let inputZip = document.getElementById("inputZip").value;

      // Use axios to make call to API
      // Read in the JSON and save the latitude and longitude into variables
      let long, lat;

      let urlQuery = `${address},+${inputCity}+${inputState}+${inputZip}`;

      // If it doesn't work try deleting params and replacing the url with

      // `https://nominatim.openstreetmap.org/search?q=${replaceSpaces(urlQuery)}&format=json`,
      const options = {
        method: 'GET',
        url: `https://nominatim.openstreetmap.org/search?q=${replaceSpaces(urlQuery)}&format=json`
      }

      axios.request(options).then(function (response) {
        console.log(response.data[0].lat)
        console.log(response.data[0].lon)
        lat = response.data[0].lat;
        long = response.data[0].lon;
        console.log("Lat: "+lat+" lon: "+long);
        addDoc(collection(db, `users/${uid}/info`), {
          firstName: firstName,
          lastName: lastName,
          email: email,
          address1: address,
          address2: address2,
          city: inputCity,
          state: inputState,
          zip: inputZip,
          latitude: lat,
          longitude: long
        }
        ).then((docRef) => {
            console.log("Document written with ID: ", docRef.id);
        });
        addDoc(collection(db, `coords`), {
          latitude: lat,
          longitude: long,
          uid: uid
        }
        ).then((docRef) => {
            console.log("Document written with ID: ", docRef.id);
            window.location.href='/home';
        });
      }).catch(function (error) {
        console.error(error);
      });



      // https://nominatim.openstreetmap.org/search?q=11th+Ave,+New+York&format=json
    }

    // replaces spaces in string with plus sign
    function replaceSpaces(str) {
        return str.replace(/\s/g, '+');
    }
</script>

<div class="mx-auto mt-5" style="width: 70%">
  <h1>{type}</h1>
  {#if type==="Login"}
    <p class="form-text">Not a user? <a href="/signup">Sign Up</a></p>
  {/if}
  <form on:submit|preventDefault={login} class="row g-3">
    {#if type==="Signup"}
      <div class="col-md-6">
          <label for="firstName" class="form-label">First Name</label>
          <input type="text" class="form-control" id="firstName" required>
      </div>
      <div class="col-md-6">
          <label for="lastName" class="form-label">Last Name</label>
          <input type="text" class="form-control" id="lastName" required>
      </div>
    {/if}
    <div class="col-md-12">
        <label for="email" class="form-label">Email address</label>
        <input type="email" class="form-control" id="email" required>
    </div>
    <div class="col-md-12">
        <label for="password" class="form-label">Password</label>
        <input type="password" class="form-control" id="password" required>
        <div id="passwordHelpBlock" class="form-text">
            Your password must be greater than 6 characters long.
        </div>
    </div>
    {#if showNotFound}
      <p class="text-danger">
          Invalid login.
      </p>
    {/if}
    {#if type==="Signup"}
      <div class="col-12">
        <label for="inputAddress" class="form-label">Address</label>
        <input type="text" class="form-control" id="inputAddress" placeholder="1234 Main St" required>
      </div>
      <div class="col-12">
        <label for="inputAddress2" class="form-label">Address 2</label>
        <input type="text" class="form-control" id="inputAddress2" placeholder="Apartment, studio, or floor">
      </div>
      <div class="col-md-6">
        <label for="inputCity" class="form-label">City</label>
        <input type="text" class="form-control" id="inputCity" required>
      </div>
      <div class="col-md-4">
        <label for="inputState" class="form-label">State</label>
        <input id="inputState" class="form-control" required>
      </div>
      <div class="col-md-2">
        <label for="inputZip" class="form-label">Zip</label>
        <input type="text" class="form-control" id="inputZip" required>
      </div>
      {/if}
    <button type="submit" class="btn btn-primary">Submit</button>
  </form>
</div>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;700&display=swap');

  h1 {
    font-family: "Open Sans", sans-serif;
    font-weight: bold;
    font-size: 3.5rem;
    background: -webkit-linear-gradient(45deg, #ff391a, #ffc4c4);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    margin-bottom: 2%;
  }

  .form-text {
    color: white;
    font-size: 1rem;
  }

  .form-text a {
    color: #ff2a2a;
  }

  label {
    color: white;
  }
</style>

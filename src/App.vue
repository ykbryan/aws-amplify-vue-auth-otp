<template>
  <div id="app">
    <b-container>
      <img alt="Vue logo" src="./assets/logo.png" />
      <h1>Welcome to AWS Amplify Demo</h1>
      <h4 v-if="message !== ''">{{ message }}</h4>
      <b-form inline v-if="user === null && session === null" class="justify-content-md-center">
        <b-form-input class="mr-2" v-model="number" placeholder="Enter phone number"></b-form-input>
        <b-button variant="outline-primary" @click="signIn">Login</b-button>
      </b-form>
      <b-form inline v-if="user === null && session !== null" class="justify-content-md-center">
        <b-form-input class="mr-2" v-model="otp" placeholder="Enter OTP " @keypress="signOut"></b-form-input>
        <b-button variant="outline-primary" @click="verifyOtp">Confirm</b-button>
      </b-form>
      <div class="mt-3">
        <b-button class="m-1" @click="verifyAuth">Verify</b-button>
        <b-button class="m-1" variant="danger" @click="signOut">Sign Out</b-button>
      </div>
    </b-container>
  </div>
</template>

<script>
// import { onMounted } from 'vue';
import Auth from "@aws-amplify/auth";

const NOTSIGNIN = "You are NOT logged in";
const SIGNEDIN = "You have logged in successfully";
const SIGNEDOUT = "You have logged out successfully";
const WAITINGFOROTP = "Enter OTP number";
const VERIFYNUMBER = "Verifying number (Country code +XX needed)";

export default {
  name: "App",
  data() {
    return {
      message: "",
      number: "",
      otp: "",
      session: null,
      user: null,
      password: Math.random().toString(10) + "Abc#"
    };
  },
  created: function() {
    let self = this;
    setTimeout(function() {
      self.verifyAuth();
    }, 3000);
  },
  methods: {
    signIn() {
      this.message = VERIFYNUMBER;
      Auth.signIn(this.number)
        .then(result => {
          this.session = result;
          this.message = WAITINGFOROTP;
        })
        .catch(e => {
          if (e.code === "UserNotFoundException") {
            this.signUp();
          } else if (e.code === "UsernameExistsException") {
            this.message = WAITINGFOROTP;
            this.signIn();
          } else {
            console.log(e.code);
            console.error(e);
          }
        });
    },
    async signUp() {
      const result = await Auth.signUp({
        username: this.number,
        password: this.password,
        attributes: {
          phone_number: this.number
        }
      }).then(() => this.signIn());
      return result;
    },
    signOut() {
      console.log("Sign Out");
      if (this.user) {
        Auth.signOut();
        this.user = null;
        this.otp = "";
        this.message = SIGNEDOUT;
      } else {
        this.message = NOTSIGNIN;
      }
    },
    verifyAuth() {
      Auth.currentAuthenticatedUser()
        .then(user => {
          this.user = user;
          this.message = SIGNEDIN;
          this.session = null;
        })
        .catch(err => {
          console.error(err);
          this.message = NOTSIGNIN;
        });
    },
    verifyOtp() {
      Auth.sendCustomChallengeAnswer(this.session, this.otp)
        .then(user => {
          this.user = user;
          this.message = SIGNEDIN;
          this.session = null;
        })
        .catch(err => {
          this.signIn();
          this.message = err.message;
          this.otp = "";
          console.log(err);
        });
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

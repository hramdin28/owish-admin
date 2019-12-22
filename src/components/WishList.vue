<template>
  <div>
    <div id="main-content" class="container">
      <div v-if="!isLogin">
        <b-form inline @submit="onLogin">
          <label class="sr-only" for="inline-form-input-username">username</label>
          <b-input
            id="inline-form-input-username"
            class="mb-2 mr-sm-2 mb-sm-0"
            placeholder="username"
            v-model="login.username"
            required
          ></b-input>

          <label class="sr-only" for="inline-form-input-username">password</label>
          <b-input-group class="mb-2 mr-sm-2 mb-sm-0">
            <b-input
              id="inline-form-input-password"
              placeholder="password"
              v-model="login.password"
              type="password"
            ></b-input>
          </b-input-group>

          <b-button type="submit" variant="primary">login</b-button>
        </b-form>
      </div>
      <div v-if="isLogin">
        <b-form inline @submit="onSubmit">
          <label class="sr-only" for="inline-form-input-name">Name</label>
          <b-input
            id="inline-form-input-name"
            class="mb-2 mr-sm-2 mb-sm-0"
            placeholder="wish"
            v-model="form.name"
            required
          ></b-input>

          <label class="sr-only" for="inline-form-input-description">Decription</label>
          <b-input-group class="mb-2 mr-sm-2 mb-sm-0">
            <b-input
              id="inline-form-input-description"
              placeholder="Decription"
              v-model="form.description"
            ></b-input>
          </b-input-group>

          <b-button type="submit" variant="primary">Save</b-button>
        </b-form>
      </div>
      <br>

      <b-list-group v-if="isLogin">
        <b-list-group-item v-for="message in this.received_messages" :key="message.id">
          <span style="float:left;">{{message.name}}</span>
          <span>{{message.description}}</span>
          <b-button
            pill
            style="float:right;"
            variant="outline-danger"
            v-on:click="deleteData(message.id)"
          >Delete</b-button>
        </b-list-group-item>
      </b-list-group>
      <br>
      <b-button v-if="isLogin" squared variant="info" v-on:click="fetchData">More</b-button>
    </div>
  </div>
</template>

<script>
import "bootstrap/dist/css/bootstrap.css";
import "bootstrap-vue/dist/bootstrap-vue.css";
import axios from "axios";
import Vue from "vue";

export default {
  name: "websocketdemo",
  data() {
    return {
      received_messages: [],
      send_message: null,
      connected: false,
      isLogin: false,
      page: -1,
      form: {
        name: "",
        description: ""
      },
      login: {
        username: "",
        password: ""
      }
    };
  },
  methods: {
    onLogin(evt) {
      evt.preventDefault();
      let postBody = {};
      postBody["username"] = this.login.username;
      postBody["password"] = this.login.password;
      axios
        .post(process.env.VUE_APP_TOKEN_URL + "authenticate", postBody)
        .then(response => {
          this.isLogin = true;
          localStorage.setItem("token", response.data.token);
          this.fetchData();
        })
        .catch(e => {
          this.isLogin = false;
          this.errors.push(e);
        });
    },
    onSubmit(evt) {
      evt.preventDefault();
      let postBody = {};
      postBody["name"] = this.form.name;
      postBody["description"] = this.form.description;

      let token = localStorage.getItem("token");
      axios
        .post(process.env.VUE_APP_BACKEND_HOST + "add", postBody, {
          headers: { Authorization: "Bearer " + token }
        })
        .then(response => {
          this.isLogin = true;
          window.location.reload();
        })
        .catch(e => {
          this.isLogin = false;
          this.errors.push(e);
        });
    },
    fetchData() {
      this.page++;
      let token = localStorage.getItem("token");
      let headers = {};
      headers["Authorization"] = "Bearer " + token;

      axios({
        method: "GET",
        url:
          process.env.VUE_APP_BACKEND_HOST +
          "fetchAllByPageNumber?pageNumber=" +
          this.page,
        headers
      }).then(
        result => {
          this.isLogin = true;
          this.received_messages.push(...Array.from(new Set(result.data)));
        },
        error => {
          this.isLogin = false;
          // eslint-disable-next-line no-console
          console.error(error);
        }
      );
    },
    deleteData(id) {
      this.page++;
      let token = localStorage.getItem("token");
      let headers = {};
      headers["Authorization"] = "Bearer " + token;
      axios({
        method: "DELETE",
        url: process.env.VUE_APP_BACKEND_HOST + "delete?id=" + id,
        headers
      }).then(
        result => {
          this.isLogin = true;
          window.location.reload();
        },
        error => {
          this.isLogin = false;
          // eslint-disable-next-line no-console
          console.error(error);
        }
      );
    }
  },
  mounted() {
    this.fetchData();
  }
};
</script>



/*
Copyright 2020 Google LLC
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    https://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

<template>
  <div>
    <div v-if="isVerified">
      <shopOwnerNavbar />
      <h1 id="heading" v-if="isVerified">My Shops</h1>
      <input type="search" v-model="searchResult" placeholder="search shops" id="searchbar" />
      <table class="table">
        <thead>
          <th>Shop Name</th>
          <th>Shop Type</th>
          <th>Active Customers</th>
          <th>Delete</th>
        </thead>
        <tbody>
          <tr v-for="shop in filteredShops" :key="shop['shop'].shopId">
            <td v-if="shop['shop'].status=='ACTIVE'">
              <a v-bind:href="'/shop-owner/shops/'+ shop['shop'].shopId" :shopId="shop['shop'].shopId">{{shop['shop'].shopName}}</a>
            </td>
            <td v-if="shop['shop'].status!='ACTIVE'">
              <p>{{shop['shop'].shopName}}</p>
            </td>
            <td>{{shop['shop'].shopType}}</td>
            <td v-if="shop['shop'].status=='ACTIVE'">
              <a v-bind:href="'/shop-owner/shops/'+ shop['shop'].shopId" :id="shop.shopId">{{shop.numberOfActiveTokens}}</a>
            </td>
            <td v-if="shop['shop'].status!='ACTIVE'">
              <p>0</p>
            </td>
            <td v-if="shop['shop'].status=='ACTIVE'">
              <button @click="deleteshop(shop['shop'].shopId)">x</button>
            </td>
            <td v-if="shop['shop'].status!='ACTIVE'">
              <button @click="restoreshop(shop['shop'].shopId)">restore</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <div v-if="isLoggedIn && !isVerified">
      <!-- TODO: ikoder - Add other features for this page -->
      <router-link to="/verify"><button class="button is-primary">Verify Me!</button></router-link>
    </div>
  </div>
</template>

<script src="https://cdn.jsdelivr.net/npm/sweetalert2@9"></script>
<script>
import shopOwnerNavbar from "../components/NavBar-ShopOwner";
import axios from "axios";
import Swal from "sweetalert2";
import firebase from "firebase";
export default {
  name: "MyShops",
  components: {
    shopOwnerNavbar
  },
  data() {
    return {
      shopList: [],
      searchResult: "",
      renderComponent: true,
      isLoggedIn: false,
      isVerified: false
    };
  },
  methods: {
    deleteshop: function(id) {
      Swal.fire({
        title: "Are you sure?",
        text: "",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, delete it!"
      }).then(result => {
        if (result.isConfirmed == true) {
          axios
            .put("http://penguin.termina.linux.test:8080/shop/", {
              shopId: id,
              shopStatus: "DELETED"
            })
            .then();
          this.renderComponent = false;
          var self = this;
          // TODO : ikoder - Add error handling in case of server error.
          this.$nextTick(() => {
            axios
              .get(
                "http://penguin.termina.linux.test:8080/shop/" +
                  firebase.auth().currentUser.uid
              )
              .then(response => (self.shopList = response.data.shops));
            self.renderComponent = true;
          });
        }
      });
    },
    restoreshop: function(id) {
      Swal.fire({
        title: "Are you sure you want to restore it?",
        text: "",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, restore it!"
      }).then(result => {
        if (result.isConfirmed == true) {
          axios
            .put("http://penguin.termina.linux.test:8080/shop/", {
              shopId: id,
              shopStatus: "ACTIVE"
            })
            .then();
          this.renderComponent = false;
          var self = this;
          // TODO : ikoder - Add error handling in case of server error.
          this.$nextTick(() => {
            axios
              .get(
                "http://penguin.termina.linux.test:8080/shop/" +
                  firebase.auth().currentUser.uid
              )
              .then(response => (self.shopList = response.data.shops));
            self.renderComponent = true;
          });
        }
      });
    }
  },
  created() {
    var user = firebase.auth().currentUser;
    if (user) {
      this.isLoggedIn = true;
      if (user.emailVerified) {
        this.isVerified = true;
      }
    }
    var self = this;
    setInterval(function() {
      axios
        .get(
          "http://penguin.termina.linux.test:8080/shop/" +
            firebase.auth().currentUser.uid
        )
        .then(response => (self.shopList = response.data.shops));
    }, 2000 /* milliseconds */);
  },
  computed: {
    filteredShops: function() {
      return this.shopList.filter(shop => {
        var result = this.searchResult.toLowerCase();
        var shopName = shop["shop"].shopName.toLowerCase();
        return shopName.match(result);
      });
    }
  }
};
</script>

<style scoped>
* {
  text-align: center;
}
button{
  margin: 2%;
}
.control {
  text-align: center;
}
#heading {
  font-size: 2rem;
}
#search {
  line-height: 2rem;
  width: 50%;
}
#searchbar {
  border-radius: 0.7em;
  border-width: 0.2em;
  line-height: 2em;
  margin-left: 2%;
  width: 40rem;
}
.table {
  width: 100%;
}
.table tbody tr td button:hover {
  background-color: aquamarine;
  cursor: pointer;
}
</style>

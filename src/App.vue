<template>
  <div id="app">
    <input type="file" @change="onChange" />
    <input type="button" @click="validate" value="Validate" />
    <span style="width: 500px">{{ msg }}</span>
    <table>
      <thead>
        <th>Invalid Format</th>
        <th>Address 1</th>
        <th>Address 2</th>
        <th>City</th>
        <th>State</th>
        <th>Zip</th>
        <th>View Map</th>
      </thead>
      <tbody>
        <tr v-for="item in addresses">
          <td><input type="text" v-model="item.err" /></td>
          <td><input type="text" v-model="item.addr1" /></td>
          <td><input type="text" v-model="item.addr2" /></td>
          <td><input type="text" v-model="item.city" /></td>
          <td><input type="text" v-model="item.state" /></td>
          <td><input type="text" v-model="item.zip" /></td>
          <td v-if="!item.err">
            <input type="button" @click="ViewAddress(item.idx)" value="View" />
          </td>
        </tr>
      </tbody>
    </table>
    <div id="map" style="height: 360px"></div>
  </div>
</template>

<script>
import readXlsxFile from "read-excel-file";
import axios from "axios";
import L from "leaflet";

export default {
  name: "App",
  data() {
    return {
      parentMessage: "Parent",
      items: [{ message: "Foo" }, { message: "Bar" }],
      addresses: [],
      map: null,
      tester: {},
      msg: "",
    };
  },
  methods: {
    onChange(event) {
      this.file = event.target.files ? event.target.files[0] : null;

      let xlsxfile = event.target.files ? event.target.files[0] : null;
      readXlsxFile(xlsxfile).then((rows) => {
        this.addresses = [];
        if (rows[0][0] === "address 1") rows.shift();
        rows.forEach((r, idx) => {
          let addr = {
            idx: idx,
            addr1: r[0] ? r[0] : "",
            addr2: r[1] ? r[1] : "",
            city: r[2] ? r[2] : "",
            state: r[3] ? r[3] : "",
            zip: r[4] ? r[4] : "",
            err: false,
          };
          this.addresses.push(addr);
        });
        this.validate();
      });
    },
    validate() {
      this.msg = "";

      this.addresses.forEach((addr) => {
        let err = false;

        if (
          addr.addr1 === "" ||
          addr.city === "" ||
          addr.state === "" ||
          addr.zip === ""
        ) {
          err = true;
        }
        if (isNaN(addr.zip) || addr.zip.toString().length !== 5) {
          err = true;
        }
        if (!addr.addr1.match(/([0-9]+\s*[A-z]+)/)) {
          err = true;
        }
        addr.err = err;
      });
    },
    ViewAddress(idx) {
      const addr = this.addresses[idx];

      let query =
        addr.addr1 +
        " " +
        addr.addr2 +
        " " +
        addr.city +
        " ," +
        addr.state +
        " " +
        addr.zip;

      var parent = this;
      const options = {
        method: "GET",
        url: "https://maptoolkit.p.rapidapi.com/geocode/search",
        params: { q: query },
        headers: {
          "X-RapidAPI-Key":
            "804dc3651emsh01929ac123cd165p11914ajsndc5f9079f906",
          "X-RapidAPI-Host": "maptoolkit.p.rapidapi.com",
        },
      };

      axios
        .request(options)
        .then(function (response) {
          if (response.data.length === 0) {
            parent.msg = "Cannot find the selected address";
          } else {
            parent.msg = "";

            if (parent.map !== null && parent.map !== undefined) {
              parent.map.remove();
            }
            parent.map = L.map("map").setView(
              [response.data[0].lat, response.data[0].lon],
              20
            );
            L.tileLayer(
              "https://maptiles.p.rapidapi.com/en/map/v1/{z}/{x}/{y}.png?rapidapi-key=804dc3651emsh01929ac123cd165p11914ajsndc5f9079f906",
              {
                attribution:
                  'Tiles &copy: <a href="https://www.maptilesapi.com/">MapTiles API</a>, Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
                maxZoom: 19,
              }
            ).addTo(parent.map);
          }
        })
        .catch(function (error) {
          console.error(error);
        });
    },
  },
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

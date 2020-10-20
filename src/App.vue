<template>
  <div class="root">
    <p v-if="loading">Loading...</p>
    <p v-if="!loading && error">
      {{ error }}
      <button @click="getLocations">Reload</button>
    </p>
    <div class="location" v-for="location in locations" :key="location.id">
      <div class="location-image">
        <img :src="location.img_url" :alt="location.name" />
      </div>
      <div class="location-desc">
        <div class="location-title">{{ location.name }}</div>
        <div class="location-type">{{ location.type }}</div>
      </div>
      <div class="location-inhabitants">
        <span v-if="location.inhabitants.length == 0" class="none"
          >No inhabitants</span
        >
        <span
          v-else
          v-for="(inhabitant, index) in location.inhabitants"
          :key="inhabitant"
        >
          <span v-if="index == location.inhabitants.length - 1">{{
            inhabitant
          }}</span>
          <span v-else>{{ inhabitant }} <span class="spd">|</span></span>
        </span>
      </div>
      <div class="notable_residents">
        <span
          v-for="notable_resident in location.notable_residents_data"
          :key="notable_resident.id"
        >
          <span
            ><img
            :title="notable_resident.name"
              class="notable_resident_img"
              :src="notable_resident.img_url"
              :alt="notable_resident.name"
          /></span>
        </span>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
export default {
  name: "LocationCharacter",
  data() {
    return {
      locations: [],
      notable_residents: [],
      loading: false,
      error: null,
    };
  },
  created() {
    this.getLocations();
  },
  methods: {
    async getLocations() {
      this.loading = true;
      try {
        let res = await axios.get(
          "https://finalspaceapi.com/api/v0/location/?limit=3"
        );
        this.locations = res.data;
        this.getNotableResident();
        this.error = null;
      } catch (err) {
        this.error = "Unable to fetch locations";
        console.log(err);
      }
      this.loading = false;
    },
    getNotableResident(locations) {
      // Loop through locations to create notable_residents_img array
      this.locations.forEach((location, index) => {
        this.locations[index].notable_residents_data = [];
      });
      let newLocations = [...this.locations];
      newLocations.forEach(async (location, index) => {
        this.locations = newLocations;
        if (location.notable_residents.length !== 0) {
          await location.notable_residents.forEach(async (resident) => {
            axios.get(resident).then((res) => {
              newLocations[index].notable_residents_data.push(res.data);
              this.locations = newLocations;
              // For some reasons Vue doesn't re-render, so this is a force re-render
              this.$forceUpdate();
            });
          });
        }
      });
    },
  },
};
</script>
<style scoped>
.root {
  margin: 0 auto;
  padding: 0 10px;
  color: #333;
  font-family: Verdana;
  font-weight: 300;
  display: flex;
  justify-content: space-around;
  flex-direction: column;
  align-items: center;
  color: purple;
}
.location {
  box-shadow: 0 5px 8px 0 rgba(0, 0, 0, 0.3);
  padding: 12px;
  margin-bottom: 10px;
  text-align: center;
  width: 18rem;
  background-color: #fafafa;
  margin-bottom: 15px;
  border-radius: 7px;
  flex-wrap: wrap;
}
.location-desc {
  display: flex;
  justify-content: space-between;
}
.location-image img {
  width: 100%;
  height: calc(16rem - 12px);
  border-radius: 7px;
  margin-bottom: 10px;
}
.location-title {
  font-size: "1.1rem";
  font-weight: 400;
}
.location-type {
  color: rgb(107, 107, 107);
  font-size: 0.9rem;
}
.location-inhabitants {
  color: rgb(107, 107, 107);
  text-align: left;
  font-size: 0.9rem;
  margin: 0.7rem 0;
}
.location-inhabitants span {
  margin: 0 0.1rem;
}
.notable_residents {
  text-align: left;
  margin: 0.7rem 0;
}
.notable_resident_img {
  width: 35px;
  height: 35px;
  border-radius: 25px;
  margin: 0.5rem 0.05rem;
}
</style>
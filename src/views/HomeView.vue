<template>
  <main class="container text-white ">
    <div class="pt-4 mb-8 relative">
      <input type="text" placeholder="Search for a city or state"
      v-model="searchQuery"
      @input="getSearchResult"
      class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">
      <ul v-if="mapboxSearchResult"
      class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
      <p class="py-2" v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>
        <p
          class="py-2"
          v-if="!searchError && mapboxSearchResult.length === 0"
        >
          No results match your query, try a different term.
        </p>
        <template v-else>
        <li v-for="searchResult in mapboxSearchResult" :key="searchResult.id"
        class="py-2 cursor-pointer"
        @click="previewCity(searchResult)"
        >
      {{ searchResult.place_name }}
      </li>
      </template>
      </ul>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

const router = useRouter()
const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(",");
  router.push({
    name: "cityView",
    params: { state: state, city: city },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    },
  });
};

 const searchQuery = ref("")
 const queryTimeout = ref(null)
 const mapboxSearchResult = ref(null)
 const searchError = ref(null)

 const mapboxAPIKEY = "pk.eyJ1IjoibS1pbmdqaSIsImEiOiJjbGZncmZ3ZWwxdmoxM3BuMWU1bXVpdDhxIn0.q5oQj5ML4rcxmEgCaBK1OA"

 const getSearchResult = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async() => {
    if(searchQuery.value !== ""){
      try{
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKEY}&types=place`)
      mapboxSearchResult.value = result.data.features
      } catch{
        searchError.value = true
      }

      return
    }
    mapboxSearchResult.value = null
  },
  300)  
}
</script>



<template>
  <v-container>
      <v-row>
        <v-col cols="1" sm="2"></v-col>
        <v-col cols="10" sm="8">
          <v-row v-if="isHeadingVisible">
            <v-col cols="12" sm="6">
              <v-row>
                <v-col cols="12">
                  <div class="display-2 font-weight-bold white--text">Unsplash</div>
                </v-col>
              </v-row>
              <v-row>
                <v-col cols="12">
                  <div class="subheading white--text">The internet's source of <a href="#" class="text-decoration-underline">freely-usable images</a>.</div>
                  <div class="subheading white--text">Powered by creators everywhere.</div>
                </v-col>
              </v-row>
            </v-col>
            <v-col cols="12" sm="6"></v-col>
          </v-row>
          <v-row>
            <v-col cols="12">
              <!-- <v-autocomplete
                v-model="searchPhrase"
                :items="autoCompleteItems"
                :solo="!isGalleryOpen"
                :dense="!isGalleryOpen"
                hide-details
                prepend-inner-icon="mdi-magnify"
                append-icon=""
                :filled="isGalleryOpen"
                :rounded="isGalleryOpen"
                :label="isGalleryOpen ? '' : initialValue"
                :value-comparator="autocompleteCheckUp"
                @keypress.enter="getPhotos">
              </v-autocomplete> -->
              <v-text-field
                :solo="!isGalleryOpen"
                hide-details
                prepend-inner-icon="mdi-magnify"
                v-model="searchPhrase"
                :filled="isGalleryOpen"
                :rounded="isGalleryOpen"
                :label="isGalleryOpen ? '' : initialValue"
                @input="autocompleteCheckUp"
                @keypress.enter="getPhotos">
              </v-text-field>
              <v-list class="mt-2" rounded dense v-if="isAutocompleteVisible">
                <v-list-item-group>
                  <v-list-item v-for="(autoCompleteItem, index) in autoCompleteItems" :key="index">
                    <v-list-item-content>
                      <v-list-item-title 
                        v-text="autoCompleteItem" 
                        @click.prevent="setSearchPhrase(autoCompleteItem)"></v-list-item-title>
                    </v-list-item-content>
                  </v-list-item>
                </v-list-item-group>
              </v-list>
            </v-col>
          </v-row>
          <v-row v-if="isTrendingVisible">
            <v-col cols="12" sm="6">
              <div class="body-2 white--text"><span class="bold">Trending:</span>
                {{ tempTrending[0] }},
                {{ tempTrending[1] }},
                {{ tempTrending[2] }}
              </div>
            </v-col>
          </v-row>
        </v-col>
        <v-col cols="1" xs="2"></v-col>
      </v-row>
    </v-container>
</template>

<script>
  import Unsplash, { toJson } from 'unsplash-js';
  import { API_KEY } from '../../Credentials.js'

  export default {
    name: 'SearchComponent',
    data(){
      return {
        isAutocompleteVisible: false,
        isHeadingVisible: true,
        isTrendingVisible: true,
        isGalleryOpen: false,
        initialValue: "Search free high-resolutions photos",
        searchPhrase: "",
        autoCompleteItems: [],
        tempTrending: ['garden','sport','coffee'],
        unsplashInstance: new Unsplash({ 
          accessKey: API_KEY.value,
          headers: {
            "Access-Control-Allow-Origin": "*"
          }
        })
      }
    },
    methods: {
      autocompleteCheckUp(){
        this.isAutocompleteVisible = false
        if(this.searchPhrase.length >= 3){
        this.isAutocompleteVisible = true
          this.unsplashInstance.search.photos(this.searchPhrase.toLowerCase())
          .then(toJson)
          .then(response => {
            response.results.forEach(responseObject => {
              responseObject.tags.forEach(tagItem => {
                if(tagItem.title.toLowerCase().includes(this.searchPhrase.toLowerCase())){
                  if(this.autoCompleteItems.length <= 0){
                    this.isAutocompleteVisible = true
                    this.autoCompleteItems.push(tagItem.title.toLowerCase())
                  } else if(this.autoCompleteItems[0] === "No records found..."){
                    this.autoCompleteItems = []
                    this.autoCompleteItems.push(tagItem.title.toLowerCase())
                  } else if(this.autoCompleteItems.length === 1){
                    this.autoCompleteItems.push(tagItem.title.toLowerCase())
                  } else {
                    let freeFromDuplicatesArray = [...new Set(this.autoCompleteItems)]
                    this.autoCompleteItems = freeFromDuplicatesArray
                  }
                } else if(this.autoCompleteItems.length <= 0) {
                  this.isAutocompleteVisible = true
                  this.autoCompleteItems.push("No records found...")
                }
              })
            })
          })
          .catch(error => console.log(error))
        }
      },
      setSearchPhrase(value){
        this.searchPhrase = value
        this.isAutocompleteVisible = false
        this.autoCompleteItems = []
        this.getPhotos()
      },
      getPhotos(){
        if(this.searchPhrase <= 3) return
        if(this.autoCompleteItems.length > 0){
          this.isAutocompleteVisible = false
          this.autoCompleteItems = []
        }
        this.isGalleryOpen = true
        this.isHeadingVisible = false
        this.isTrendingVisible = false
        this.$eventBus.$emit('hide-appBar')
        this.unsplashInstance.search.photos(this.searchPhrase)
        .then(toJson)
        .then(response => {
          this.$eventBus.$emit('gallery-items', { response: response, searchPhrase: this.searchPhrase })
        })
        .catch(error => console.log(error))
      }
    }
  }
</script>

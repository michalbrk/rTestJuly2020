<template>
    <v-container v-if="isGalleryVisible">
        <v-container v-if="isResultsNavVisible">
            <v-row fluid>
                <v-col cols="12">
                    <div class="display-1 font-weight-bold black--text">{{ searchPhrase }}</div>
                </v-col>
            </v-row>
            <v-row>
                <v-col cols="12">
                    <v-card flat max-width="90vw">
                        <v-card-actions class="text-center">
                            <v-chip-group show-arrows v-for="(allTag, index) in allTags" :key="index">
                                <v-chip label outlined class="grey--text">{{ allTag }}</v-chip>
                            </v-chip-group>
                        </v-card-actions>
                    </v-card>
                </v-col>
            </v-row>
        </v-container>
        <v-container v-if="isGalleryGridVisible" grid-list-lg>
            <v-layout wrap>
                <v-flex xs12 sm6 md4 v-for="image in images" :key="image.id">
                    <v-card flat tile>
                        <v-responsive>
                            <v-img 
                                :src="image.previewURL" 
                                :key="image.UID"
                                @click.prevent="displayModalWindow(
                                                        image.regularURL,
                                                        image.user.profileImage,
                                                        image.user.name,
                                                        image.user.socialMedia,
                                                        image.location
                                                        )"></v-img>
                        </v-responsive>
                        <v-card-actions>
                            <v-chip-group v-for="(tag, index) in image.tags" :key="index">
                                <v-chip label>{{ tag }}</v-chip>
                            </v-chip-group>
                        </v-card-actions>
                    </v-card>
                </v-flex>
            </v-layout>
        </v-container>
        <v-container v-if="isModalWindowVisible">
            <v-overlay :value="overlay">
                <v-card color="white" max-height="95vh" max-width="90vw">
                    <v-card-actions>
                        <v-list-item>
                            <v-list-item-avatar>
                                <v-img :src="modalWindow.userProfileImage"></v-img>
                            </v-list-item-avatar>
                            <v-list-item-content>
                                <v-list-item-title class="headline black--text" v-text="modalWindow.userName"></v-list-item-title>
                                <v-list-item-subtitle class="grey--text" v-text="modalWindow.userSocialMedia"></v-list-item-subtitle>
                            </v-list-item-content>
                        </v-list-item>
                        <v-spacer></v-spacer>
                        <v-btn small color="white grey--text">
                            <v-icon>mdi-heart</v-icon>
                        </v-btn>
                        <v-btn small color="white grey--text">
                            <v-icon>mdi-plus</v-icon>
                        </v-btn>
                    </v-card-actions>
                    <v-card-actions>
                        <v-container>
                            <v-layout>
                                <v-flex xs0 sm1 md2></v-flex>
                                <v-flex xs12 sm10 md8>
                                    <v-card>
                                        <v-img cover :src="modalWindow.regularImage" max-height="60vh"></v-img>
                                    </v-card> 
                                </v-flex>
                                <v-flex xs0 sm1 md2></v-flex>
                            </v-layout>
                        </v-container>
                    </v-card-actions>
                    <v-card-actions>
                        <v-list-item>
                            <v-list-item-icon>
                                <v-icon class="grey--text">mdi-heart</v-icon>
                            </v-list-item-icon>
                            <v-list-item-content>
                                <v-list-item-subtitle class="black--text" v-text="modalWindow.location"></v-list-item-subtitle>
                            </v-list-item-content>
                        </v-list-item>
                        <v-spacer></v-spacer>
                        <v-btn small color="white grey--text"><v-icon>mdi-share</v-icon>Share</v-btn>
                        <v-btn small color="white grey--text"><v-icon>mdi-info</v-icon>Info</v-btn>
                        <v-btn small color="white blue--text" @click.prevent="closeModalWindow">Close</v-btn>
                    </v-card-actions>
                </v-card>
            </v-overlay>
        </v-container>
    </v-container>
</template>
<script>
export default {
    name: 'GalleryComponent',
    data(){
        return {
            overlay: false,
            isGalleryVisible: false,
            isResultsNavVisible: true,
            isGalleryGridVisible: true,
            isModalWindowVisible: false,
            images: [],
            allTags: [],
            searchPhrase: "",
            modalWindow: {
                regularImage: "",
                userProfileImage: "",
                userName: "",
                userSocialMedia: "",
                location: ""
            }
        }
    },
    mounted(){
        this.$eventBus.$on('gallery-items', (responseFromSearch) => {
            this.images = []
            responseFromSearch.response.results.forEach(imageObject => {
                this.images.push({
                    UID: imageObject.id,
                    previewURL: imageObject.urls.small,
                    regularURL: imageObject.urls.regular,
                    tags: [
                        imageObject.tags[0].title,
                        imageObject.tags[1].title,
                        imageObject.tags[2].title,
                    ],
                    user: {
                        profileImage: imageObject.user.profile_image.small,
                        name: imageObject.user.name,
                        socialMedia: `@${imageObject.user.instagram_username}`
                    },
                    location: 'Unknown...'
                })
                imageObject.tags.forEach(tagItem => {
                    this.allTags.push(tagItem.title)
                })
            })
            this.isGalleryVisible = true
            this.searchPhrase = responseFromSearch.searchPhrase
        })
    },
    methods: {
        displayModalWindow(...values){
            this.isModalWindowVisible = true
            this.overlay = true
            let self = this
            function* propsGenerator(){
                for(let propertyName of Object.keys(self.modalWindow)){
                    yield propertyName
                }
                return
            }
            let propsGeneratorObject = propsGenerator()

            values.forEach(propertyValue => {
                Object.defineProperty(
                    this.modalWindow, 
                    propsGeneratorObject.next().value, 
                    { value: propertyValue })
            })
        },
        closeModalWindow(){
            this.isModalWindowVisible = false
            this.overlay = false
        }
    }
}
</script>
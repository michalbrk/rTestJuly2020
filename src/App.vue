<template>
<div :class="{ galleryHidden: isGalleryHidden, galleryVisible: isGalleryVisible }">
  <v-app id="transparent">
    <v-app-bar v-if="isAppBarVisible" flat max-height="20vh" color="transparent"></v-app-bar>
    <SearchComponent/>
    <GalleryComponent/>
  </v-app>
</div>
</template>

<script>
import SearchComponent from './components/SearchComponent'
import GalleryComponent from './components/GalleryComponent'

export default {
  name: 'App',
  components: {
    SearchComponent,
    GalleryComponent
  },
  data(){
    return {
      isAppBarVisible: true,
      isGalleryHidden: true,
      isGalleryVisible: false
    }
  },
  mounted(){
    this.$eventBus.$on('hide-appBar', () => {
      this.isAppBarVisible = false
    })
    this.$eventBus.$on('gallery-items', () => {
      this.isGalleryHidden = false
      this.isGalleryVisible = true
    })
  },
  beforeDestroy(){
    this.$eventBus.$off('hide-appBar')
    this.$eventBus.$off('gallery-items')
  }
};
</script>
<style lang="scss">
@mixin nobackground {
  background: none;
}
@mixin background {
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}
.galleryHidden {
  background-image: url('assets/sunset.jpg');
  @include background;
}
.galleryVisible {
  @include nobackground;
}
#transparent {
  @include nobackground;
}
</style>
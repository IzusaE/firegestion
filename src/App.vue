<template>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-rbsA2VBKQhggwzxH7pPCaAqO46MgnOM80zW1RWuH61DGLwZJEdK2Kadq2F9CUG65" crossorigin="anonymous"> 

  <nav class="navbar navbar-expand-lg">
      <a class="navbar-brand" href="./home.html"><img src="./assets/carapuce.png"></a>
      <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
      </button>
      
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
          <ul class="navbar-nav mr-auto">
              <li class="nav-item active">
                  <a class="nav-link" href="#/">Map</a>
              </li>
              <li class="nav-item">
                  <a class="nav-link" href="#/caserne">Casernes</a>
              </li>
              <li class="nav-item">
                  <a class="nav-link" href="#/pompier">Pompiers</a>
              </li>
          </ul>
      </div>
  </nav>
  <component :is="currentView" />
</template>

<script>
import MapComponent from './components/MapComponent.vue'
import CaserneComponent from './components/CaserneComponent.vue'
import PompiersComponent from './components/PompiersComponent.vue'
import NotFound from './components/NotFound.vue'

const routes = {
    '/': MapComponent,
    '/caserne': CaserneComponent,
    '/pompier': PompiersComponent
}

export default {
  name: 'App',
  data() {
    return {
        currentPath: window.location.hash
    }
  },
  computed: {
    currentView() {
        return routes[this.currentPath.slice(1) || '/'] || NotFound
    }
  },
  mounted() {
    window.addEventListener('hashchange', () => {
        this.currentPath = window.location.hash
    })
  }
}
</script>
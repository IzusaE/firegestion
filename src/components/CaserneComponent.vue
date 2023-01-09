<template>
  <template v-if="loader">
    <div id="centerLoader">
        <div class="spinner-grow text-success" role="status">
            <span class="sr-only"></span>
        </div>
    </div>
  </template>
  <template v-else>
    <div id="carserneContaineur">
        <button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#caserne">
            Nouvelle caserne
        </button>     
        <div class="containCasernes">
            <div class="oneCaserne" v-for="caserne in casernes" :key="caserne.id">
                <div class="titleCaserne">
                    <div class="textCaserne">
                        {{caserne.name}}<br>
                        <button type="button" class="btn btn-primary btnTitle" data-bs-toggle="modal" :data-bs-target="'#caserne' + caserne.id">
                            Nouveau Camion
                        </button> 
                        <div class="modal fade" :id="'caserne' + caserne.id" tabindex="-1" :aria-labelledby="'caserne' + caserne.id" aria-hidden="true">
                            <div class="modal-dialog">
                                <div class="modal-content">
                                    <div class="modal-header">
                                        <h1 class="modal-title fs-5" id="exampleModalLabel">Nouveau Camion pour {{caserne.name}}</h1>
                                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                                    </div>
                                    <div class="modal-body">
                                        <div class="mb-3">
                                            <label for="exampleInputPassword1" class="form-label">Latitude</label>
                                            <input type="text" v-model="latNewCamion" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" required>
                                        </div>
                                        <div class="mb-3">
                                            <label for="exampleInputPassword1" class="form-label">Longitude</label>
                                            <input type="text" v-model="lonNewCamion" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" required>
                                        </div>
                                        <div class="mb-3">
                                            <label for="exampleInputPassword1" class="form-label">Pompier n°1</label>
                                            <select v-model="equipier1">
                                                <option v-for="pomp in pompiers" :key="pomp.id" :value="pomp.id">{{pomp.first_name}} {{pomp.last_name}}</option>
                                            </select>
                                        </div>
                                        <div class="mb-3">
                                            <label for="exampleInputPassword1" class="form-label">Pompier n°2</label>
                                            <select v-model="equipier2">
                                                <option v-for="pomp in pompiers" :key="pomp.id" :value="pomp.id">{{pomp.first_name}} {{pomp.last_name}}</option>
                                            </select>
                                        </div>
                                    </div>
                                    <div class="modal-footer">
                                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Annuler</button>
                                        <button type="button" class="btn btn-success" @click="createNewCamion(caserne.id)">Créer</button>
                                    </div>
                                </div>
                            </div>
                        </div> 
                    </div>
                    <div class="blueLine"></div>
                </div>
                <div class="containCamions">
                    <div class="myCamion" v-for="camion in listeCamions" :key="camion.id">
                        <div class="titreCamion">
                            {{camion.name}}
                        </div>
                        <div class="equipeCamion">
                            <img v-for="(equipier, index) in camion.equipe" :key="index" :src="equipier" />
                        </div>
                    </div>
                </div>                   
            </div>
        </div>        
    </div>

    <!-- Nouvelle caserne modal -->
    <div class="modal fade" id="caserne" tabindex="-1" aria-labelledby="caserne" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h1 class="modal-title fs-5" id="exampleModalLabel">Nouvelle Caserne</h1>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <form>
                        <div class="mb-3">
                            <label for="exampleInputEmail1" class="form-label">Nom de la caserne</label>
                            <input type="text" v-model="nomCaserne" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" required>
                        </div>
                        <div class="mb-3">
                            <label for="exampleInputPassword1" class="form-label">Latitude</label>
                            <input type="text" v-model="latNewCaserne" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" required>
                        </div>
                        <div class="mb-3">
                            <label for="exampleInputPassword1" class="form-label">Longitude</label>
                            <input type="text" v-model="lonNewCaserne" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Annuler</button>
                    <button type="button" class="btn btn-success" @click="createNewCaserne()">Créer</button>
                </div>
            </div>
        </div>
    </div>
  </template>
</template>

<script>

export default {
    components: {
    },
    data() {
        return {
            loader: false,
            camions: [],
            casernes: [],
            pompiers: [],
            types: [],
            listeSprites: [],
            nomCaserne: "",
            lonNewCaserne: "",
            latNewCaserne: "",
            lonNewCamion: "",
            latNewCamion: "",
            equipier1: "",
            equipier2: "",
        };
    },
    async beforeMount() {
        this.loader = true

        this.camions = await this.makeRequest("GET", "/api/firetruck");
        this.casernes = await this.makeRequest("GET", "/api/firestation");
        this.pompiers = await this.makeRequest("GET", "/api/fireman")
        this.types = await this.makeRequest("GET", "https://pokebuildapi.fr/api/v1/types")
        this.listeSprites = await this.makeRequest("GET", "https://pokebuildapi.fr/api/v1/pokemon")

        this.camions.forEach(element => {
            var equipe = []
            this.pompiers.forEach(elem => {
                if (element.id == elem.firetruck_id)
                    equipe.push(this.listeSprites[Math.floor(Math.random() * 600)].sprite)
            })
            element.equipe = equipe
        });
        console.log(this.camions)
        this.loader = false
    }, 
    computed: {
        listeCamions() {
            return this.camions
        }
    },
    methods: {
        makeRequest(method, url, body) {
            return new Promise(function (resolve, reject) {
                let xhr = new XMLHttpRequest();
                xhr.open(method, url);
                xhr.onload = function () {
                    if (this.status >= 200 && this.status < 300) {
                        resolve(JSON.parse(xhr.response));
                    } else {
                        reject({
                            status: this.status,
                            statusText: xhr.statusText
                        });
                    }
                };
                xhr.onerror = function () {
                    reject({
                        status: this.status,
                        statusText: xhr.statusText
                    });
                };
                xhr.send(body);
            });
        },
        async createNewCaserne() {
            var msg = new Object()
            msg.name = this.nomCaserne
            msg.coords = [this.latNewCaserne, this.lonNewCaserne]

            this.makeRequest("POST", "/api/firetruck", msg)
            
            this.casernes = await this.makeRequest("GET", "/api/firestation");
        },
        async createNewCamion(id) {
            var msg = new Object()

            msg.firestation_id = id
            msg.coords = [this.latNewCamion, this.lonNewCamion]

            var newCamion = this.makeRequest("POST", "/api/firetruck", msg)
            
            this.camions = await this.makeRequest("GET", "/api/firetruck");

            var equipier1 = new Object()
            var equipier2 = new Object()

            this.pompiers.forEach(element => {
                if (element.id == this.equipier1) {
                    equipier1.firestation_id = newCamion.firestation_id
                    equipier1.firetruck_id = newCamion.id
                    equipier1.first_name = newCamion.first_name
                    equipier1.last_name = newCamion.last_name
                }
                if (element.id == this.equipier2) {
                    equipier2.firestation_id = newCamion.firestation_id
                    equipier2.firetruck_id = newCamion.id
                    equipier2.first_name = newCamion.first_name
                    equipier2.last_name = newCamion.last_name
                }
            });

            await this.makeRequest("POST", `/api/fireman/${this.equipier1}/update`, equipier1)
            await this.makeRequest("POST", `/api/fireman/${this.equipier2}/update`, equipier2)

        }
    }
};
</script>
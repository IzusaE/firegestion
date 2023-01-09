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
        <div class="containCasernes">
            <div class="oneCaserne" v-for="caserne in casernes" :key="caserne.id">
                <div class="titleCaserne">
                    <div class="textCaserne">
                        {{caserne.name}}<br>
                        <button type="button" class="btn btn-primary btnTitle" data-bs-toggle="modal" :data-bs-target="'#caserne' + caserne.id">
                            Nouveau Pompier
                        </button> 
                        <div class="modal fade" :id="'caserne' + caserne.id" tabindex="-1" :aria-labelledby="'caserne' + caserne.id" aria-hidden="true">
                            <div class="modal-dialog">
                                <div class="modal-content">
                                    <div class="modal-header">
                                        <h1 class="modal-title fs-5" id="exampleModalLabel">Nouveau pompier pour {{caserne.name}}</h1>
                                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                                    </div>
                                    <div class="modal-body">
                                        <div class="mb-3">
                                            <label for="exampleInputEmail1" class="form-label">Prénom du pompier</label>
                                            <input type="text" v-model="lastName" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" required>
                                        </div>
                                        <div class="mb-3">
                                            <label for="exampleInputEmail1" class="form-label">Nom du pompier</label>
                                            <input type="text" v-model="firstName" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" required>
                                        </div>
                                    </div>
                                    <div class="modal-footer">
                                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Annuler</button>
                                        <button type="button" class="btn btn-success" @click="nouveauPompier(caserne.id)">Créer</button>
                                    </div>
                                </div>
                            </div>
                        </div> 
                    </div>
                    <div class="blueLine"></div>
                </div>
                <div class="containCamions">
                    <div class="myCamion" v-for="pompier in listePompiers" :key="pompier.id">
                        <div class="titreCamion">
                            {{pompier.last_name}} {{pompier.first_name}}
                        </div>
                    </div>
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
            lastName: "",
            firstName: "",
        };
    },
    async beforeMount() {
        this.loader = true


        this.camions = await this.makeRequest("GET", "/api/firetruck");
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
        listePompiers() {
            return this.pompiers
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
        async nouveauPompier(id) {
            var msg = new Object()
            msg.first_name = this.firstName
            msg.last_name = this.lastName
            msg.firestation_id = id

            this.makeRequest("POST", "/api/fireman", msg)

            this.pompiers = await this.makeRequest("GET", "/api/fireman")
        }
    }
};
</script>
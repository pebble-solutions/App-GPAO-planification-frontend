<template>
    <AppModal id="ajoutGroup" title="Ajout Groupé" :pending="pending.ajoutgroupe" :submit-btn="true" :cancel-btn="true" @submit="addToGroup()" @modal-hide="backPreviousRoute()">
        <div v-if="projetsList">
            <div class="form-group">
                <label for="selectmetier">Choissisez un des projets affichés:</label>

                <select class="form-select" id="selectprojet" name="projet" v-model="selectedProjetId">
                    <option v-for="(projet) in projetsList" :value="projet.id" :key="projet.id">{{projet.intitule}}</option>
                </select>
            </div>

            <hr>  
            
            <div class="form-group" >
                <label for="selectmetier">Choissisez un métier:</label>

                <AlertMessage v-if="!selectedProjet" class="mt-3">Sélectionnez un projet avant de pouvoir renseigner un metiers</AlertMessage>

                <select v-else class="form-select" id="selectmetier" name="projet__liaison_besoin_rh_id" v-model="ressources_besoin_group.projet__liaison_besoin_rh_id">
                    <option v-for="(metier) in selectedProjet.besoins_rh" :value="metier.id" :key="metier.id">{{metier.oType.nom}} ({{metier.periode_code}})</option>
                </select>
            </div>

            <hr v-if="selectedProjet">  
    
            
            <div class="form-row" v-if="selectedProjet">
                <div class="form-group col">
                    <label for="datedebut">date début:</label>
                    <input type="date" class="form-control" id="datedebut" name="dd" :min="selectedProjet.ddp" :max="selectedProjet.dfp" v-model="ressources_besoin_group.dd" required>
                </div>
    
                <div class="form-group col">
                    <label for="datefin">date fin:</label>
                    <input type="date" class="form-control" id="datefin" name="df" :min="selectedProjet.ddp" :max="selectedProjet.dfp" v-model="ressources_besoin_group.df" required>
                </div>

                <div class="from-group col">
                    <label for="quantity">Nombre:</label>
                    <input type="number" class="form-control" name="nb" id="quantity" v-model="ressources_besoin_group.nb" required>
                </div>
            </div> 
            
            <hr>  
            
            <!--jours de semaine-->
            <h3>Jours</h3>
            
            <div class="form-row">
                <div class="col" v-for="(gp, k) in days" :key="k">
                    <div v-for="(i, day) in gp" :key="day">
                        <div class="form-check form-switch">
                            <input type="checkbox" name="days[]" class="form-check-input" role="switch" :id="day" value="1" v-model="ressources_besoin_group.days[i]">
                            <label class="form-check-label" :for="day">{{day}}</label>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </AppModal>
</template>

<script>
import AppModal from '@/components/pebble-ui/AppModal.vue';
import AlertMessage from '@/components/pebble-ui/AlertMessage.vue'

import { mapActions, mapState } from 'vuex';

let BESOIN_GROUP_PROPS = {
    dd: null,
    df: null,
    nb: 0,
    projet_id: null,
    projet__liaison_besoin_rh_id: null,
    days: [false, false, false, false, false, false, false]
}

export default {
    props: {
        ajoutGroupe : Boolean,
    },

    data() {
        return {
            days: [{lundi:0,mardi:1, mercredi:2, jeudi:3, vendredi:4}, {samedi:5, dimanche:6}],
            pending: {
                ajoutgroupe: false
            },
            routeProjetListId: null,
            selectedProjetId: null
        }
    },

    beforeRouteEnter(to, from, next) {
        next(vm => {
            vm.routeProjetListId = from.params.projetListId;
        })
    },

    computed: {
        ...mapState(['projetsList', 'ressourcesRHType', 'selectedCells']),

        /**
         * Initialise la variable ressources_besoin_group avec le mappage de BESOIN_GROUP_PROPS
         */
        ressources_besoin_group() {
            return this.initGroupModal();
        },

        /**
         * Récupère un projet dans projetsList en fonction de l'id sélectionné
         */
        selectedProjet() {
            return this.projetsList.find(projet => projet.id == this.selectedProjetId);
        }
    },

    components: {AppModal, AlertMessage},

    methods: {
        ...mapActions(['refreshRessourcesBesoin']),
        
        /**
         * initGroupModal()
         * Initialise une copie des données par défaut lors de la création par groupe. Les 
         * données par défaut sont stockées dans la constante BESOIN_GROUP_PROPS et dupliquées 
         * sur la variable locale ressources_besoin_group.
         * À la fin de l'initialisation la boite modale est appelée.
         */
        initGroupModal() {
            let ressources_besoin_group = {};

            for (let key in BESOIN_GROUP_PROPS) {
                ressources_besoin_group[key] = BESOIN_GROUP_PROPS[key];
            }

            return ressources_besoin_group;
        },

        /**
         * Retourne a la page précédente
         */
         backPreviousRoute() {
            this.$router.push({name:"Ressources", params: {projetListId: this.routeProjetListId}});
        },

        /**
         * addToGroup()
         * Prépare une requête pour un groupe de données à ajouter dans les besoins. Cette fonction est appelée
         * après le remplissage du formulaire d'ajout groupé.
         * La requête est préparée puis envoyée dans la méthod recordGroup()
         */
        addToGroup() {
            let daySelected = '';

            for (let i = 0; i < this.ressources_besoin_group.days.length; i++) {
                if (this.ressources_besoin_group.days[i]) {
                    if ('' === daySelected) {
                        daySelected += (i+1)
                    } else {
                        daySelected += ',' + (i+1)
                    }
                }
            }


            let query = {
                dd: this.ressources_besoin_group.dd,
                df: this.ressources_besoin_group.df,
                nb: this.ressources_besoin_group.nb,
                projet_id: this.selectedProjet.id,
                projet__liaison_besoin_rh_id: this.ressources_besoin_group.projet__liaison_besoin_rh_id,
                days: daySelected
            }

            this.recordGroup(query);
        },

        /**
         * Enregistre les valeurs du formulaire de l'ajout groupé des ressourcesBesoin et les enregistre dans le store
         * 1. injecter l'objet nouvellement créé en POST via la clé query
         * 2. traiter le retour
         * 
         * @param {Object} query               les options a envoyer a l'api
         * 
         * @return {Promise}
         */
        recordGroup(query) {
            this.pending.ajoutgroupe = true;

            let newRessourcesBesoin = [];
            let urlApi = "/ressourcesBesoin/POST/group";

            this.$app.apiPost(urlApi, 
                query
            ).then((data) => {
                newRessourcesBesoin['action'] = 'update'
                newRessourcesBesoin['data'] = data

                this.refreshRessourcesBesoin(newRessourcesBesoin);
                
                this.pending.ajoutGroup = false;
                this.backPreviousRoute();
            }).catch(this.$app.catchError);
        },
    },
}
</script>
<template>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css"
  />

  <table id="tablefacturier" @click="editFacturier">
    <thead id="theadTableauFacturier">
      <tr>
        <th>Actions</th>
        <th>N°</th>
        <th>Apprenti</th>
        <th>Formation</th>
        <th>Employeur</th>
        <th>Contrat</th>
        <th>OPCO</th>
        <th>Reste</th>
        <th>Factures</th>
        <th>Etape</th>
        <th></th>
      </tr>
    </thead>
    <tbody id="tbodyfiltresFacturier">
      <tr>
        <td></td>
        <td id="premiereCaseTableauRecherche">
          <form id="formcreadossier">
            <input type="hidden" name="cdate" value="" />
            <input type="hidden" name="cerfa.etat" value="0" />
            <div class="boutonCreationDossier" @click="creerDossier">
              <span class="iconeAjout"
                ><font-awesome-icon icon="fa-person-circle-plus"
              /></span>
              <span>Nouveau dossier</span>
            </div>
          </form>
        </td>
        <td>
          <select>
            <option>Choisir</option>
          </select>
        </td>
        <td>
          <select>
            <option>Choisir</option>
            <option>CI</option>
            <option>CDUI</option>
            <option>COM</option>
            <option>GPME</option>
            <option>MCO</option>
            <option>MMV</option>
            <option>NDRC</option>
            <option>SAM</option>
          </select>
        </td>
        <td>
          <select>
            <option>Choisir</option>
          </select>

          <!--bouton-base @click="formMaitre = true" class="BoutonBaseRecherche" id="BoutonBaseRechercheMaitre" :intituleBouton="this.$data.nomBoutonMaitre" v-on:click="this.ajouterUnMaitre"></bouton-base-->
        </td>
        <td></td>
        <td>
          <select>
            <option value="0">Choisir</option>
            <option v-for="objet in opcos" :value="objet._id.$oid">
              {{ objet.nom }}
            </option>
          </select>
        </td>
        <td></td>
        <td></td>
        <td>
          <select>
            <option>en cours</option>
          </select>
        </td>
        <td></td>
      </tr>
    </tbody>
    <tbody>
      <tr>
        <td colspan="10">
          <FormulaireOpco
            v-if="etatFormulaire == 'opco'"
            v-on:remetEtatInitial="this.remetEtatInitial"
            id="formOpco"
            @insertion-bdd="insereObjetDansBdd"
          >
          </FormulaireOpco>
          <FormulaireApprenti
            v-if="etatFormulaire == 'cerfa.apprenti'"
            v-on:remetEtatInitial="this.remetEtatInitial"
            :objetInit="itemEdite"
            id="formApprenti"
          >
          </FormulaireApprenti>
        </td>
      </tr>
    </tbody>
    <tbody id="lignesDuFacturier">
      <tr
        v-for="(item, index) in itemsAffiches"
        :data-num="index + nbItemsParPage * pageCourante"
      >
        <td>
          <i
            class="fa fa-arrow-right"
            aria-hidden="true"
            @click="transmettreDossier"
          ></i
          ><i
            class="fa fa-trash-o"
            aria-hidden="true"
            @click="mettreDossierCorbeille"
          ></i>
        </td>
        <td>
          {{
            item.cerfa.numeroExterne || item.cerfa.numeroInterne || 'NOUVEAU'
          }}
        </td>
        <td class="editable" data-prop="cerfa.apprenti">
          <span v-if="item.cerfa.apprenti"
            >{{ item.cerfa.apprenti.prenom }}
            {{ item.cerfa.apprenti.nom }}</span
          >
          <font-awesome-icon
            v-else
            @click="$emit(this.evenement)"
            class="fontIcone"
            icon="fa-file-circle-plus"
          />
        </td>
        <td>
          <span v-if="item.cerfa.formation"
            >{{ item.cerfa.formation.intituleQualification }}
          </span>
          <font-awesome-icon
            v-else
            @click="$emit(this.evenement)"
            class="fontIcone"
            icon="fa-file-circle-plus"
          />
        </td>
        <td>
          <span v-if="item.cerfa.employeur">{{
            item.cerfa.employeur.denomination
          }}</span>
          <font-awesome-icon
            v-else
            @click="$emit(this.evenement)"
            class="fontIcone"
            icon="fa-file-circle-plus"
          />
        </td>
        <td>
          <span v-if="item.cerfa.contrat">{{
            formateDate(item.cerfa.contrat.dateDebutContrat)
          }}</span>
        </td>
        <td>{{ item.opco }}</td>
        <td>{{ resteAPayer(item.echeances) }}</td>
        <td>
          <span
            :class="'echeance echeance-' + etatEcheance(echeance)"
            v-for="echeance in item.echeances"
            >{{ resteAPayer(echeance) }}</span
          >
        </td>
        <td></td>
        <td>{{ item.cerfa.etat }}</td>
      </tr>
    </tbody>
  </table>
  <MiseAJourService @ongetliste="miseAJour"></MiseAJourService>
  <div class="navtable">
    <button
      class="changepage"
      :disabled="this.pageCourante === 0"
      @click="changePage(false)"
    >
      &larr;
    </button>
    <span> {{ pageCourante + 1 }} / {{ nbTotalPages }}</span>
    <button
      class="changepage"
      :disabled="this.pageCourante + 1 >= nbTotalPages"
      @click="changePage"
    >
      &rarr;
    </button>
  </div>
</template>

<script>
import MiseAJourService from '@/services/MiseAJourService.vue';
import BoutonBase from '@/components/Controler/elementsHTML/bouton/BoutonBase.vue';
import elementContratTableauFacturier from '@/components/Controler/backOffice/elementContratTableauFacturier.vue';
import FormulaireApprenti from '@/components/Controler/backOffice/FormulaireApprenti.vue';
import FormulaireOpco from '@/components/Controler/backOffice/FormulaireOpco.vue';
import formulaireEmployeur from '@/components/Controler/backOffice/formulaireEmployeur.vue';
import formulaireMaitre from '@/components/Controler/backOffice/formulaireMaitre.vue';
//import formulaireContrat from "@/components/Controler/backOffice/formulaireContrat.vue";
import construitURLService from '@/services/construitURL.service.vue';
import configuration from '@/administration/configuration.vue';
import connexionAPIService from '@/services/connexionAPI.service.vue';

const ECHEANCE_ETAT_INIT = 'initial';
const ECHEANCE_ETAT_REGLE = 'regle';
const ECHEANCE_ETAT_RETARD = 'retard';
const ECHEANCE_ETAT_ENCOURS = 'en_cours';
const ECHEANCE_DELAI_JOURS_PAIEMENT = 3;

export default {
  name: 'tableauFactureNonSoldees',
  components: {
    BoutonBase,
    FormulaireOpco,
    FormulaireApprenti,
    elementContratTableauFacturier,
    MiseAJourService,
  },
  data() {
    return {
      typeBtnAfficherFormulaire: 'ouvrirformulaire',
      etatFormulaire: '',
      items: [],
      nomCollectionPrincipale: 'dossier',
      opcos: [],
      apprentis: [],
      employeurs: [],
      dossiers: [],
      nbItemsParPage: 10,
      pageCourante: 0,
      itemEdite: null,
      idCourant: 0,
    };
  },
  computed: {
    itemsAffiches() {
      return this.items.slice(
        this.pageCourante * this.nbItemsParPage,
        this.pageCourante * this.nbItemsParPage + this.nbItemsParPage
      );
    },
    nbTotalPages() {
      return Math.ceil(this.items.length / this.nbItemsParPage);
    },
  },
  methods: {
    editFacturier(e) {
      let form = e.currentTarget,
        t = e.target;
      e.preventDefault();
      if (t.classList && t.classList.contains('editable')) {
        let numItem = parseInt(t.parentNode.getAttribute('data-num')),
          prop = t.getAttribute('data-prop'),
          item = this.items[numItem];
        if (item) {
          let p = prop.split('.');
          this.itemEdite = p.reduce(function (a, c) {
            return a[c];
          }, item);
          this.idCourant =
            typeof item._id == 'string'
              ? item._id
              : "ObjectId('" + item._id.$oid + ")'";
          this.changeEtaFormulaire(prop);
        }
      }
    },
    changePage(next = true) {
      this.pageCourante += next ? 1 : -1;
    },
    creerDossier(e) {
      let form = document.getElementById('formcreadossier'),
        url = construitURLService.methods.construitURLConnectionBack(
          'dossier',
          configuration.data().urlPossibles.ajouter
        );
      form.elements.cdate.value = Date.now();
      connexionAPIService.methods.requete(url, form).then((reponse) => {
        if (reponse.code_reponse !== 0) {
        } else {
        }
      });
    },
    etatEcheance(echeance) {
      let r = this.resteAPayer(echeance),
        d = echeance.dateOuverture,
        dt = new Date(d);
      if (r == 0) {
        return ECHEANCE_ETAT_REGLE;
      }
      if (dt.getTime() > Date.now) {
        return ECHEANCE_ETAT_INIT;
      }
      if (
        dt.getTime() + ECHEANCE_DELAI_JOURS_PAIEMENT * 86400000 <
        Date.now()
      ) {
        return ECHEANCE_ETAT_RETARD;
      }
      return ECHEANCE_ETAT_ENCOURS;
    },
    resteAPayer(echeances) {
      if (!echeances) {
        return 0;
      }
      if (!Array.isArray(echeances)) {
        echeances = [echeances];
      }
      let r = echeances.reduce(function (a, c) {
        return (
          a + parseFloat(c.montantTotal || 0) - parseFloat(c.montantRegle || 0)
        );
      }, 0);
      r = r.toFixed(2);
      return r;
    },
    formateDate(d) {
      let fd = new Date(d);
      return fd.toLocaleDateString();
    },
    miseAJour(nomCollection, liste) {
      if (nomCollection != this.nomCollectionPrincipale) {
        this[nomCollection + 's'] = liste;
      } else {
        liste.forEach((d) => {
          if (d.echeances) {
            d.echeances.sort((a, b) => {
              return;
              +(a.dateOuverture > b.dateOuverture) ||
                +(a.dateOuverture == b.dateOuverture) - 1;
            });
          }
        });
        liste.sort((a, b) => {
          let _a = a.cdate || 0,
            _b = b.cdate || 0;
          return +(_a < _b) || +(_a == _b) - 1;
        });
        this.items = liste;
        let apprentis = liste.reduce((a, c) => {
          if (c.cerfa.apprenti) {
            a.push(c.cerfa.apprenti.prenom + ' ' + c.cerfa.apprenti.nom);
          }
          return a;
        }, []);
      }
    },
    changeEtaFormulaire(etat) {
      if (this.etatFormulaire == etat) {
        this.etatFormulaire = '';
      } else {
        this.etatFormulaire = etat;
      }
    },
    resetFormulaire(NomFormulaire) {
      let formulaire = document.getElementById('lesFormulairesAjoutFacturier');
      let barreRechercheFacturier = document.getElementById('filtresFacturier');
      formulaire.classList.add('fermetureFormulaireFacturier');
      barreRechercheFacturier.style.height = '4rem';
      this.$data['nomBouton' + NomFormulaire] = 'ajouter';
      this['form' + nomFormulaire] = false;
    },
    apprentiMineurEmancipe() {
      let barreRechercheFacturier = document.getElementById('filtresFacturier');
      barreRechercheFacturier.style.height = '28rem';
    },
    apprentiMineurNonEmancipe() {
      let barreRechercheFacturier = document.getElementById('filtresFacturier');
      barreRechercheFacturier.style.height = '35rem';
    },
    apprentiMajeur() {
      let barreRechercheFacturier = document.getElementById('filtresFacturier');
      barreRechercheFacturier.style.height = '26rem';
    },
    ajouterUneSection() {
      console.log('ajoute une section');
    },
    remetEtatInitial() {
      console.log('remet en place');
      this.resetFormulaireApprenti();
    },
    async insereObjetDansBdd(nomCollection, json) {
      let URL = construitURLService.methods.construitURLConnectionBack(
        nomCollection,
        configuration.data().urlPossibles.ajouter
      );
      await connexionAPIService.methods
        .requete(URL, json)
        .then((reponseBDD) => {
          if (reponseBDD.code_reponse !== 0) {
            alert('erreur insereObjetDansBdd : ' + reponseBDD.Error_info);
          } else {
            json._id = reponseBDD.extra_info;
            this.$emit('insertionObjetOk', nomCollection, json);
            console.log(
              'Objet ajouté en base de données, collection : ' +
                nomCollection +
                ', _id:' +
                json._id
            );
          }
        });
    },
    async effacerFormulaire() {
      for (let valeur of document.getElementsByClassName('detailOpco')[0]
        .children) {
        valeur.lastChild.value = '';
      }
    },
    mettreDossierCorbeille(e) {
      console.log(this.items);
      console.log(e);
      console.log(e.parentNode);
      console.log(e.parentElement);
    },
    transmettreDossier() {},
  },
};
</script>

<style scoped>
#titreTableauFacturerNonSolde {
  display: flex;
  background: var(--mauve-clair);
  height: 28px;
  box-shadow: 0px 5px 5px -3px;
}

.titreTableau h4 {
  font-weight: 300;
  font-size: 1rem;
  color: white;
  text-align: center;
}

#numeroTitreFacturier {
  width: 3%;
  min-width: 30px;
  background: var(--color-dark);
  margin-right: 1px;
  border-radius: 0 0 6px 0;
}

#titreTableauFacturerNonSolde :nth-child(2) {
  background: var(--color-dark);
  width: 18%;
  min-width: 150px;
  margin-right: 1px;
  border-radius: 6px 0 6px 0;
}

#titreTableauFacturerNonSolde :nth-child(3) {
  width: 5%;
  min-width: 60px;
  background: var(--color-dark);
  margin-right: 1px;
  border-radius: 6px 0 6px 0;
}
#titreTableauFacturerNonSolde :nth-child(4) {
  width: 18%;
  min-width: 100px;
  background: var(--color-dark);
  margin-right: 1px;
  border-radius: 6px 0 6px 0;
}

#titreTableauFacturerNonSolde :nth-child(5) {
  width: 6%;
  min-width: 60px;
  background: var(--color-dark);
  margin-right: 1px;
  border-radius: 6px 0 6px 0;
}

#titreTableauFacturerNonSolde :nth-child(6) {
  width: 8%;
  min-width: 70px;
  background: var(--color-dark);
  margin-right: 1px;
  border-radius: 6px 0 6px 0;
}

#titreTableauFacturerNonSolde :nth-child(7) {
  width: 8%;
  min-width: 70px;
  background: var(--color-dark);
  margin-right: 1px;
  border-radius: 6px 0 6px 0;
}

#titreTableauFacturerNonSolde :nth-child(8) {
  width: 8%;
  min-width: 98px;
  background: var(--color-dark);
  margin-right: 1px;
  border-radius: 6px 0 6px 0;
}

#titreTableauFacturerNonSolde :nth-child(9) {
  width: 25%;
  min-width: 150px;
  background: var(--color-dark);
  border-radius: 6px 0 0 0;
}

#titreTableauDerniereCase {
  width: 15%;
  min-width: 130px;
  background: var(--color-dark);
  background: var(--color-dark);
  height: 28px;
}

/* Valeur height a changer dynamiquement pour englober les formulaires*/
#tbodyfiltresFacturier {
  background: var(--mauve-clair);
  height: 4rem;
  box-shadow: 0px 5px 5px -4px;
  transition-property: height;
  transition-duration: 1s;
}
#tbodyfiltresFacturier tr {
  background: #c0c9d8;
}
#tablefacturier thead tr th,
#tbodyfiltresFacturier tr td {
  text-align: center;
}
#tbodyfiltresFacturier .fontIcone {
  margin: 3px;
}
.enteteRecherche {
  height: 3.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

select {
  width: 100%;
}

#lesFormulairesAjoutFacturier {
  display: none;
  position: absolute;
  top: 9.5rem;
  margin-top: 4em;
  width: 100%;
  height: 13rem;
  animation: fadein 3s;
}

.OuvertureFormulaireFacturier {
  -webkit-transition: 2s;
  -moz-transition: 2s;
  -ms-transition: 2s;
  -o-transition: 2s;
  transition: 2s;
  margin-left: 0%;
  opacity: 1;
}

.fermetureFormulaireFacturier {
  -webkit-transition: 2s;
  -moz-transition: 2s;
  -ms-transition: 2s;
  -o-transition: 2s;
  transition: 2s;
  margin-left: 100%;
  opacity: 0;
}
@keyframes fadein {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.boutonCreationDossier {
  margin: 0.5rem 1rem 0.5rem 1rem;
  padding: 0.3rem;
  text-align: center;
  border-style: dashed;
  color: #ffaece;
  border-color: #ffaece;
  text-transform: capitalize;
  border-radius: 10px;
  transition-duration: 1s;
  background: white;
  line-height: 16px;
  text-align: center;
}

.iconeAjout {
  font-size: 120%;
}

.boutonCreationDossier span {
  margin-left: 0.5rem;
  margin-right: 0.5rem;
  display: inline-block;
}

.boutonCreationDossier:hover {
  font-size: 120%;
  cursor: pointer;
}

#lignesDuFacturier {
  /*
  La seule barre de defilement du facturier
   */
  background: var(--mauve-clair);
  flex-grow: 3;
  margin: 0rem 0.5rem 0.5rem 0.3rem;
  margin-bottom: 2.4rem;
  border-radius: 7px;
  box-shadow: 0px 5px 5px -3px;
}
#lignesDuFacturier tr:nth-child(odd) {
  background: white;
}
.detailApprenti select {
  width: 50%;
}
#etatmiseajour {
  position: fixed;
  bottom: 3px;
  right: 3px;
  background: #80d0f0;
  color: blue;
  padding: 2px;
  font-size: 10px;
  font-weight: bold;
  display: block;
}
.echeance {
  display: inline-block;
  padding: 1px 4px;
  border-radius: 6px;
  background: white;
  font-size: 11px;
}
.echeance-retard {
  background: red;
  color: white;
}
.echeance-regle {
  background: greenyellow;
  color: green;
}
.echeance-en_cours {
  background: #eaeaff;
  color: black;
}
.navtable {
  position: absolute;
  bottom: 5px;
  width: 100%;
  text-align: center;
}
.navtable span {
  background: white;
}
.navtable button.changepage {
  display: inline-block;
  width: 40px;
  margin: 5px;
}
.editable:hover {
  cursor: pointer;
  color: white;
  background: blue;
}
.editable {
  background: inherit;
}
.editable * {
  pointer-events: none;
}
form.formulaireAjoutFacturier {
  border: dashed #ffaece 7px;
  border-radius: 20px;
}
</style>

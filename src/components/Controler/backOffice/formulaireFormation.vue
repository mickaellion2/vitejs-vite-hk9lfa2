<template>
  <form :action="action" class="formulaireAjoutFacturier">
    <fieldset class="titreFormulaireFacturier">
      <legend>
        Formulaire Formation (<a
          target="_blank"
          href="https://www.formulaires.service-public.fr/gf/getNotice.do?cerfaNotice=51649&cerfaFormulaire=10103"
          style="font-weight: bold"
          >Aide</a
        >)
      </legend>
      <input type="hidden" name="__base__" value="cerfa.formation" />
      <input type="hidden" name="__id__" value="0" />
      <div class="detailFormation">
        <div class="inputBoxFacturier">
          <span class="detailFacturier">RNCP</span>
          <input
            type="text"
            placeholder="obligatoire"
            maxlength="9"
            name="rncp"
            required
          />
        </div>
        <div class="inputBoxFacturier">
          <span class="detailFacturier"
            >Code du dipl&ocirc;me ou titre visé</span
          >
          <input type="text" name="codeDiplome" maxlength="8" />
        </div>

        <div class="inputBoxFacturier">
          <span class="detailFacturier"
            >Dipl&ocirc;mes et titres de l'Formation</span
          >
          <select name="typeDiplome">
            <optgroup label="Dipl&ocirc;me ou titre de niveau bac +5 et plus">
              <option value="80" label="Doctorat"></option>
              <option value="71" label="Master professionnel/DESS"></option>
              <option value="72" label="Master recherche/DEA"></option>
              <option value="73" label="Master indifférencié"></option>
              <option
                value="74"
                label="Dipl&ocirc;me d'ingénieur, dipl&ocirc;me d'école de commerce"
              ></option>
              <option
                value="79"
                label="Autre dipl&ocirc;me ou titre de niveau bac+5 ou plus"
              ></option>
            </optgroup>
            <optgroup label="Dipl&ocirc;me ou titre de niveau bac +3 et 4 ">
              <option value="61" label="1ère année de Master"></option>
              <option value="62" label="Licence professionnelle"></option>
              <option value="63" label="Licence générale"></option>
              <option
                value="69"
                label="Autre dipl&ocirc;me ou titre de niveau bac +3 ou 4"
              ></option>
            </optgroup>
            <optgroup label="Dipl&ocirc;me ou titre de niveau bac +2">
              <option
                value="54"
                label="Brevet de Technicien Supérieur"
              ></option>
              <option
                value="55"
                label="Dipl&ocirc;me Universitaire de technologie"
              ></option>
              <option
                value="58"
                label="Autre dipl&ocirc;me ou titre de niveau bac+2"
              ></option>
            </optgroup>
            <optgroup label="Dipl&ocirc;me ou titre de niveau bac">
              <option value="41" label="Baccalauréat professionnel"></option>
              <option value="42" label="Baccalauréat général"></option>
              <option value="43" label="Baccalauréat technologique"></option>
              <option
                value="49"
                label="Autre dipl&ocirc;me ou titre de niveau bac"
              ></option>
            </optgroup>
            <optgroup label="Dipl&ocirc;me ou titre de niveau CAP/BEP">
              <option value="33" label="CAP"></option>
              <option value="34" label="BEP"></option>
              <option value="35" label="Mention complémentaire"></option>
              <option
                value="38"
                label="Autre dipl&ocirc;me ou titre de niveau CAP/BEP"
              ></option>
            </optgroup>
            <optgroup label="Aucun dipl&ocirc;me ni titre">
              <option
                value="25"
                label="Dipl&ocirc;me national du Brevet"
              ></option>
              <option
                value="26"
                label="Certificat de formation générale"
              ></option>
              <option
                value="13"
                label="Aucun dipl&ocirc;me ni titre professionnel"
              ></option>
            </optgroup>
          </select>
        </div>

        <div class="inputBoxFacturier">
          <span class="detailFacturier"
            >Date de début du cycle de formatio</span
          >
          <input type="date" name="dateDebutFormation" required />
        </div>

        <div class="inputBoxFacturier">
          <span class="detailFacturier">Date de fin du cycle de formatio</span>
          <input type="date" name="dateDebutFormation" required />
        </div>

        <div class="inputBoxFacturier">
          <span class="detailFacturier">Intitulé précis du dipl&ocirc;me</span>
          <input
            type="text"
            name="intituleQualification"
            minlength="1"
            maxlength="255"
            placeholder="obligatoire"
            required
          />
        </div>
      </div>

      <BoutonBase
        class="BoutonBaseRecherche"
        intituleBouton="effacer"
        v-on:click="this.effacerFormulaire"
      ></BoutonBase>

      <BoutonBase
        class="BoutonBaseRecherche"
        intituleBouton="Soumettre"
      ></BoutonBase>
      <div v-if="afficheErreurs" class="erreur">
        <p>{{ messageErreur }}</p>
      </div>
    </fieldset>
  </form>
</template>

<script>
import BoutonBase from '@/components/Controler/elementsHTML/bouton/BoutonBase.vue';
import construitURLService from '@/services/construitURL.service.vue';
//import {Formation} from "@/components/Model/Formation.Class";
//import {Formation} from "@/components/Model/FormationTS.Class";
import Formation from '@/components/Model/FormationJS.Class';
import creationJSONService from '@/services/creationJSON.service.vue';
import configuration from '@/administration/configuration.vue';
import connexionAPIService from '@/services/connexionAPI.service.vue';

export default {
  name: 'FormulaireFormation',
  components: {
    BoutonBase,
  },
  props: {},
  data() {
    return {
      action: construitURLService.methods.construitURLConnectionBack(
        'dossier',
        configuration.data().urlPossibles.modifier
      ),
      listeDepartements: [],
      afficheFormulaireMineur: false,
      afficheMineurNonEmancipe: true,
      afficheErreurs: true,
      messageErreur: '',
      listeGeographie: [],
      tableauRecherche: [],
    };
  },
  mounted() {
    var _this = this;
    if (
      document.querySelector('#departementsNaissanceListe').childElementCount <=
      1
    ) {
      this.getListeDepartements().then((d) => {
        _this.init();
      });
    } else {
      this.init();
    }
  },
  methods: {
    init() {
      let obj = this.$parent.itemEdite,
        form = this.$el,
        _id_ = this.$parent.idCourant;
      form.elements['__id__'].value = _id_;
      if (obj) {
        for (let inputElt of form.elements) {
          let prop = inputElt.name;
          if (prop) {
            if (prop == '__id__') {
              continue;
            }
            let props = prop.split('.'),
              v;
            v =
              props.reduce(function (a, c) {
                return a[c];
              }, obj) || '';
            if (inputElt.type == 'checkbox') {
              inputElt.checked = v == 1 || v == 'true';
            } else {
              inputElt.value = v;
            }
          }
        }
      }
    },
    beforeSubmit() {},
    SiFormationMineur(event) {
      console.log(event);
      //on recupere la date du jour
      const dateDuJour = Date.now();
      //On recuperer la saisie de l'agent et on soustrait 18 ans
      let dateDeNaissance = document.getElementById(
        'dateNaissanceFormation'
      ).value;
      this.$data.dateNaissance = dateDeNaissance;
      dateDeNaissance = dateDeNaissance.split('-');
      dateDeNaissance = new Date(
        dateDeNaissance[0],
        dateDeNaissance[1] - 1,
        dateDeNaissance[2]
      );
      dateDeNaissance = dateDeNaissance.getTime();
      const dixHuitAnsEnMillisecondes = 567648000000;
      if (dateDuJour - dateDeNaissance < dixHuitAnsEnMillisecondes) {
        //On affiche la partie pour les mineurs du formulaire
        this.$data.afficheFormulaireMineur = true;
        //On passe l'information au composant parent
        this.$emit('afficheMineurNonEmancipe', event);
      } else {
        this.$data.afficheFormulaireMineur = false;
        this.$emit('afficheMajeur', event);
      }
    },
    SiMineurEmancipe(event) {
      if (this.$data.afficheMineurNonEmancipe) {
        this.$data.afficheMineurNonEmancipe = false;
        this.$emit('afficheMineurEmancipe', event);
      } else {
        this.$data.afficheMineurNonEmancipe = true;
        this.$emit('afficheMineurNonEmancipe', event);
      }
    },
    async getListeDepartements() {
      var _this = this;
      return fetch('https://geo.api.gouv.fr/departements')
        .then((response) => response.json())
        .then((data) => {
          _this.listeDepartements = data;
          return data;
        });
    },
    getListeCommunesNaissance() {
      let v;
      if ((v = document.querySelector('#departementsNaissanceListe').value)) {
        d = fetch('https://geo.api.gouv.fr/departements/' + v + '/communes')
          .then((response) => response.json())
          .then((data) => data.forEach((element) => {}));
      }
    },
    rechercheCommunes() {
      return true;
    },
    async rentrerFormationBDD(event) {
      console.log(this.$data.nom_de_naissance);
      if (true || document.querySelector('.formulaire').reportValidity()) {
        console.log('Les informations du formulaire sont valides !');
        let Formation = new Formation(
          this.$data.nom_de_naissance,
          this.$data.nom_usage,
          this.$data.prenom,
          this.$data.dateNaissance,
          this.$data.sexe,
          this.$data.commune_de_naissance,
          this.$data.departement_de_naissance,
          this.$data.adresse,
          this.$data.complement_adresse,
          this.$data.code_postal,
          this.$data.commune,
          this.$data.nationalite,
          this.$data.telephone,
          this.$data.courriel,
          this.$data.travailleur_handicape,
          this.$data.numero_de_securite_sociale,
          this.$data.cle_numero_de_securite_sociale,
          this.$data.mineur_emancipe,
          this.$data.nom_du_representant_legal,
          this.$data.adresse_du_representant_legal,
          this.$data.code_postal_du_representant_legal,
          this.$data.prenom_du_representant_legal,
          this.$data.complement_adresse_du_representant_legal,
          this.$data.commune_du_representant_legal
        );
        let JSON = creationJSONService.methods.construitJSON(Formation);
        let URL = construitURLService.methods.construitURLConnectionBack(
          configuration.data().collections.Formation,
          configuration.data().urlPossibles.ajouter
        );
        //Reservation case mémoire de la BDD
        let reponseBDD = await connexionAPIService.methods.requete(URL, JSON);
        if (reponseBDD.code_reponse !== 0) {
          alert('erreur : ' + reponseBDD.Error_info);
        } else {
          Formation.modifieIdentifiantFormation(
            reponseBDD.extra_info.identifiantFormation
          );
          JSON = creationJSONService.methods.construitJSON(Formation);
          //Modification de la case mémoire et ajout de l'Formation en BDD
          URL = construitURLService.methods.construitURLConnectionBack(
            configuration.data().collections.Formation,
            configuration.data().urlPossibles.modifier
          );
          reponseBDD = await connexionAPIService.methods.requete(URL, JSON);
          if (reponseBDD.code_reponse !== 0) {
            alert('erreur : ' + reponseBDD.Error_info);
          } else {
            this.$emit('remetEtatInitial', event);
            alert('Formation ajouté en base de données');
          }
        }
      } else {
        console.log(
          'Les informations du formulaire sont incorrectes, pas de liaison avec la BDD !'
        );
      }
    },
    async effacerFormulaire() {
      for (let valeur of document.getElementsByClassName('detailFormation')[0]
        .children) {
        /*console.log(valeur.lastChild.value);*/
        valeur.lastChild.value = '';
      }
    },
  },
};
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Poppins', sans-serif;
}

/*section{
      display: flex;
      padding: 10px;
    }*/

.detailFormation {
  width: 100%;
  max-width: content-box;
  border-radius: 5px;
  padding: 10px;
  background: white;
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  /*grid-template-columns: repeat(5, 1fr);
        grid-gap: 0.2rem 0.5rem;*/
  box-shadow: 0px 5px 5px -5px;
  justify-content: flex-start;
  align-items: flex-start;
}

.etudiantMineur:last-child {
  max-width: content-box;
  border-radius: 5px;
  padding: 10px;
  background: white;
  display: flex;
  grid-template-columns: repeat(3, 1fr);
  grid-gap: 0.2rem 0.5rem;
}

.detailFormationMineur .etudiantMineur div {
  margin: auto;
}

.ligneHorizontaleFormulaire {
  margin: 1em;
  height: 3px;
  background: linear-gradient(#e66465, #9198e5);
  width: 95%;
}

.titreFormulaireFacturier {
  text-align: center;
}

.inputBoxFacturier {
  display: flex;
  flex-direction: column;
  margin-right: auto;
  width: inherit;
  flex-basis: 20%;
  margin-right: 1%;
  margin-top: 1px;
  margin-bottom: 1px;
  margin-left: 1%;
  align-items: flex-start;
  justify-content: center;
}

select {
  width: inherit;
}

option {
  data-width: 100%;
}

.formulaire {
  width: 90%;
}
</style>

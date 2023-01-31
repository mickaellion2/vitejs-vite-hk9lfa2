<template>
  <form :action="action" class="formulaireAjoutFacturier">
    <fieldset class="titreFormulaireFacturier">
      <legend>
        Formulaire Maitre (<a
          target="_blank"
          href="https://www.formulaires.service-public.fr/gf/getNotice.do?cerfaNotice=51649&cerfaFormulaire=10103"
          style="font-weight: bold"
          >Aide</a
        >)
      </legend>
      <input type="hidden" name="__base__" value="cerfa.maitre1" />
      <input type="hidden" name="__id__" value="0" />
      <div class="detailApprenti">
        <div class="inputBoxFacturier">
          <span class="detailFacturier">Nom de naissance</span>
          <input
            type="text"
            placeholder="obligatoire"
            minlength="1"
            maxlength="80"
            name="nom"
            required
          />
        </div>
        <div class="inputBoxFacturier">
          <span class="detailFacturier">Prénom</span>
          <input
            type="text"
            name="prenom"
            placeholder="obligatoire"
            minlength="1"
            maxlength="80"
            required
          />
        </div>
        <div class="inputBoxFacturier">
          <span class="detailFacturier">Date de naissance</span>
          <input
            id="dateNaissanceApprenti"
            type="date"
            name="dateNaissance"
            @input="this.SiApprentiMineur"
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

  <form :action="action" class="formulaireAjoutFacturier">
    <fieldset class="titreFormulaireFacturier">
      <legend>
        Formulaire Maitre (<a
          target="_blank"
          href="https://www.formulaires.service-public.fr/gf/getNotice.do?cerfaNotice=51649&cerfaFormulaire=10103"
          style="font-weight: bold"
          >Aide</a
        >)
      </legend>
      <input type="hidden" name="__base__" value="cerfa.maitre2" />
      <input type="hidden" name="__id__" value="0" />
      <div class="detailApprenti">
        <div class="inputBoxFacturier">
          <span class="detailFacturier">Nom de naissance</span>
          <input
            type="text"
            placeholder="obligatoire"
            minlength="1"
            maxlength="80"
            name="nom"
            required
          />
        </div>
        <div class="inputBoxFacturier">
          <span class="detailFacturier">Prénom</span>
          <input
            type="text"
            name="prenom"
            placeholder="obligatoire"
            minlength="1"
            maxlength="80"
            required
          />
        </div>
        <div class="inputBoxFacturier">
          <span class="detailFacturier">Date de naissance</span>
          <input
            id="dateNaissanceApprenti"
            type="date"
            name="dateNaissance"
            @input="this.SiApprentiMineur"
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
//import {Maitre} from "@/components/Model/Maitre.Class";
//import {Maitre} from "@/components/Model/MaitreTS.Class";
import Maitre from '@/components/Model/MaitreJS.Class';
import creationJSONService from '@/services/creationJSON.service.vue';
import configuration from '@/administration/configuration.vue';
import connexionAPIService from '@/services/connexionAPI.service.vue';

export default {
  name: 'formulaireMaitre',
  components: {
    BoutonBase,
  },

  data() {
    return {
      action: construitURLService.methods.construitURLConnectionBack(
        'dossier',
        configuration.data().urlPossibles.modifier
      ),
    };
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
    async rentrerMaitresBDD(event) {
      console.log(this.$data.nom_de_naissance);

      let maitreUn = new Maitre(
        this.$data.nom_un,
        this.$data.prenom_un,
        this.$data.dateNaissance_un,
        this.$data.attestation_un
      );
      let maitreDeux = new Maitre(
        this.$data.nom_deux,
        this.$data.prenom_deux,
        this.$data.dateNaissance_deux,
        this.$data.attestation_deux
      );

      let JSON = creationJSONService.methods.construitJSON(maitreUn);
      let URL = construitURLService.methods.construitURLConnectionBack(
        configuration.data().collections.maitreUn,
        configuration.data().urlPossibles.ajouter
      );
      //Reservation case mémoire de la BDD
      let reponseBDD = await connexionAPIService.methods.requete(URL, JSON);
      if (reponseBDD.code_reponse !== 0) {
        alert('erreur : ' + reponseBDD.Error_info);
      } else {
        maitreUn.modifieIdentifiantMaitre(
          reponseBDD.extra_info.identifiantMaitre
        );
        JSON = creationJSONService.methods.construitJSON(maitreUn);
        //Modification de la case mémoire et ajout de l'maitreUn en BDD
        URL = construitURLService.methods.construitURLConnectionBack(
          configuration.data().collections.maitreUn,
          configuration.data().urlPossibles.modifier
        );
        reponseBDD = await connexionAPIService.methods.requete(URL, JSON);
        if (reponseBDD.code_reponse !== 0) {
          alert('erreur : ' + reponseBDD.Error_info);
        } else {
          this.$emit('remetEtatInitial', event);
          alert('Maitre ajouté en base de données');
        }
      }

      let JSON2 = creationJSONService.methods.construitJSON(maitreDeux);
      let URL2 = construitURLService.methods.construitURLConnectionBack(
        configuration.data().collections.maitreDeux,
        configuration.data().urlPossibles.ajouter
      );
      //Reservation case mémoire de la BDD
      let reponseBDD2 = await connexionAPIService.methods.requete(URL2, JSON2);
      if (reponseBDD2.code_reponse !== 0) {
        alert('erreur : ' + reponseBDD2.Error_info);
      } else {
        maitreDeux.modifieIdentifiantMaitre(
          reponseBDD2.extra_info.identifiantMaitre
        );
        JSON2 = creationJSONService.methods.construitJSON(maitreDeux);
        //Modification de la case mémoire et ajout de l'maitreDeux en BDD
        URL2 = construitURLService.methods.construitURLConnectionBack(
          configuration.data().collections.maitreDeux,
          configuration.data().urlPossibles.modifier
        );
        reponseBDD2 = await connexionAPIService.methods.requete(URL2, JSON2);
        if (reponseBDD2.code_reponse !== 0) {
          alert('erreur : ' + reponseBDD2.Error_info);
        } else {
          this.$emit('remetEtatInitial', event);
          alert('Maitre ajouté en base de données');
        }
      }
    },
    async effacerFormulaire() {
      for (let valeur of document.getElementsByClassName('detailMaitre')[0]
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

.detailMaitre {
  width: 90%;
  max-width: content-box;
  border-radius: 5px;
  padding: 10px;
  background: white;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 0.2rem 0.5rem;
  box-shadow: 0px 5px 5px -5px;
}

.detailMaitreMineur {
  width: 90%;
  max-width: content-box;
  border-radius: 5px;
  padding: 10px;
  background: white;
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-gap: 0.2rem 0.5rem;
  box-shadow: 0px 5px 5px -5px;
}

.etudiantMineur {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.detailMaitreMineur .etudiantMineur div {
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
}

/*.detailMaitre div:last-child{
    display: grid;
    grid-template-columns: repeat(3, 1fr);
}*/
</style>

/* Suggestions : -prévoir d'utiliser une API pour rentrer des données
géographiques -prévoir d'utiliser une API pour rentrer des données d'entreprise
-faire une validation des données afin de vérifier quelles sont bien dans le bon
format */

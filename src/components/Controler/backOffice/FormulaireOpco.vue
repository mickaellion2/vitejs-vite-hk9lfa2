<template>
  <form id="formid" :action="action" class="formulaireAjoutFacturier">
    <legend class="titreFormulaireFacturier">
      <h5>
        Formulaire Opco
        <span class="boutonAide" @click="afficheaide = !afficheaide"
          >(Aide)</span
        >
      </h5>
    </legend>
    <div v-if="afficheaide">
      Remplir les informations sur l'Opco puis les identifiants pour les
      accrochages en recette et en production.
    </div>
    <fieldset>
      <div class="detailOpco">
        <div>
          <div id="opcoInfos">
            <span class="detailFacturier">Informations</span>
            <div class="inputBoxFacturier">
              <span class="detailFacturier">Nom</span>
              <input
                type="text"
                name="nom"
                placeholder="obligatoire"
                required
              />
            </div>
            ....
          </div>
          <div id="opcoAccrochages">
            <span class="detailFacturier">Accrochages</span>
            <div class="opcoAccrochage">
              <span class="detailFacturier">Accrochage Recette</span>
              <div class="inputBoxFacturier">
                <input type="hidden" name="accrochages.0.prod" value="0" />
                <span class="detailFacturier">Clé API Recette</span>
                <input
                  class="key"
                  type="text"
                  name="accrochages.0.apikey"
                  placeholder="obligatoire"
                  required
                />
              </div>
              <div class="inputBoxFacturier">
                <span class="detailFacturier">Client id recette</span>
                <input
                  class="key"
                  type="text"
                  name="accrochages.0.clientid"
                  placeholder="obligatoire"
                  required
                />
              </div>
              <div class="inputBoxFacturier">
                <span class="detailFacturier">Client secret recette</span>
                <input
                  class="key"
                  type="text"
                  name="accrochages.0.clientsecret"
                  placeholder="obligatoire"
                  required
                />
              </div>
              <div class="inputBoxFacturier">
                <span class="detailFacturier">Token Url recette</span>
                <input
                  class="url"
                  type="text"
                  name="accrochages.0.tokenurl"
                  placeholder="obligatoire"
                  required
                />
              </div>
              <div class="inputBoxFacturier">
                <span class="detailFacturier">Api Url recette</span>
                <input
                  class="url"
                  type="text"
                  name="accrochages.0.apiurl"
                  placeholder="obligatoire"
                  required
                />
              </div>
            </div>
          </div>
        </div>
        <BoutonBase
          class="BoutonBaseRecherche"
          intituleBouton="effacer"
          @click="this.effacerFormulaire"
        ></BoutonBase>
        <BoutonSubmit
          @on-espace-submit-success="opcoOk"
        ></BoutonSubmit>
      </div>
    </fieldset>
  </form>
</template>

<script>
import BoutonBase from '@/components/Controler/elementsHTML/bouton/BoutonBase.vue';
import BoutonSubmit from '@/components/Controler/elementsHTML/bouton/BoutonSubmit.vue';

import construitURLService from "@/services/construitURL.service.vue";
import configuration from '@/administration/configuration.vue';

export default {
  name: 'FormulaireOpco',
  components: {
    BoutonBase,
    BoutonSubmit,
  },
  data() {
    return {
      action: construitURLService.methods.construitURLConnectionBack("opco", configuration.data().urlPossibles.ajouter),
    };
  },
  methods: {
    opcoOk() {},
  },
};
</script>
<style>
.boutonAide {
  font-size: 0.8em;
}
.boutonAide:hover {
  cursor: pointer;
}
</style>
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

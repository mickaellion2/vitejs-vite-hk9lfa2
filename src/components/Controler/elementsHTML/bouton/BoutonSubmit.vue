<template>
  <BoutonBase
    :intituleBouton="intitule"
    :evenementBouton="evenement"
    @click="onSubmit"
  ></BoutonBase>
</template>

<script>
import BoutonBase from '@/components/Controler/elementsHTML/bouton/BoutonBase.vue';
import connexionAPIService from '@/services/connexionAPI.service.vue';
import construitURLService from '@/services/construitURL.service.vue';
import configuration from '@/administration/configuration.vue';

export default {
  name: 'BoutonSubmit',
  components: {
    BoutonBase,
  },
  props: {
    intituleBouton: String,
  },
  data() {
    return {
      intitule: this.intituleBouton || 'Soumettre',
      param: this.nomCollection,
      evenement: 'onEspaceSubmit',
    };
  },
  methods: {
    onSubmit(e) {
      e.preventDefault();
      /*if (this.$parent.methods.beforeSubmit) {
        this.$parent.methods.beforeSubmit();
      }*/
      var _this = this;
      let form = this.$el.form;
      if (form.reportValidity()) {
        connexionAPIService.methods
          .requete(form.action, form)
          .then((reponse) => {
            if (reponse.code_reponse !== 0) {
              console.log('erreur code reponse');
              _this.$emit('onEspaceSubmitFail', reponse);
            } else {
              _this.$emit('onEspaceSubmitSuccess', reponse);
            }
          });
      }
    },
  },
};
</script>

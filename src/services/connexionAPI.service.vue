<script>
export default {
  name: 'connexionAPI.service',
  methods: {
    async requete(url, form) {
      let f = form ? form : new FormData();
      f = new FormData(form);
      return  await fetch(url, {
          method: 'POST',
          body: f
        })
        .then(async (reponse) => {
          if (!reponse.ok) {
            throw new Error(`HTTP erreur. status : ${reponse.status}`);
          }
          return reponse.json();
        })
        .catch((e) => {
          throw new Error('Attention : ' + e.message);
        });
    },
  },
};
</script>

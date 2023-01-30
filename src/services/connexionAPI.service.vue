<script>
const ErreurHTML = '';
export default {
  name: 'connexionAPI.service',
  methods: {
    async requete(url, form) {
      let f = form ? new FormData(form) : new FormData();
      return await fetch(url, {
        method: 'POST',
        body: f,
      })
        .then(async (reponse) => {
          if (!reponse.ok) {
            throw new Error(`HTTP erreur. status : ${reponse.status}`);
          }
          if (true) {
            ErreurHTML = reponse.text();
            document.body.append(t);
          }
          return reponse.json();
        })
        .catch((e) => {
          if (ErreurHTML) {
            document.body.append(ErreurHTML);
            ErreurHTML = '';
          }
          throw new Error('Attention : ' + e.message);
        });
    },
  },
};
</script>

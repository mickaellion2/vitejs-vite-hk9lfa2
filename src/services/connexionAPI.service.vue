<script>
var ErreurHTML = '';
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
          /*if (true) {
            //ErreurHTML = reponse.text();
            let r = reponse.body.tee();
            ErreurHTML = r[0].text();
          }
          return r[1].json();*/
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

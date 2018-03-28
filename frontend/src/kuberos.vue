<template>
  <div id="kuberos">
    <b-alert v-if="error" show variant="danger">
      <p>Authentication failed: {{error.response.status}} {{error.response.statusText}}: {{error.response.data}}</p>
    </b-alert>
    <b-container v-else fluid>
      <b-row><br /></b-row>
      <b-row>
        <b-col></b-col>
        <b-col md="10">
          <b-jumbotron>
            <template slot="header">Kuberos</template>
          </b-jumbotron>
        </b-col>
        <b-col></b-col>
      </b-row>

      <b-row>
        <b-col></b-col>
        <b-col md="10">
          <h3>Authenticate</h3>
          <p>
          The following will configure authentication for your @ada.support user to the Kubernetes cluster.
          </p>
          <pre v-highlightjs="snippetSetCreds()"><code class="bash"></code></pre>
        </b-col>
        <b-col></b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
export default {
  name: "kuberos",
  metaInfo: {
    title: "Kubernetes Authentication",
    htmlAttrs: {
      lang: "en"
    }
  },
  data: function() {
    return {
      error: null,
      kubecfg: {}
    };
  },
  methods: {
    templateURL: function() {
      return "kubecfg.yaml?" + $.param(this.kubecfg);
    },
    snippetSetCreds: function() {
      return (
        "# Add your user to kubectl\n" +
        'kubectl config set-credentials "' +
        'user@' + window.location.hostname.split('.').slice(-2).join('.') +
        '" \\\n' +
        "  --auth-provider=oidc \\\n" +
        '  --auth-provider-arg=client-id="' +
        this.kubecfg.clientID +
        '" \\\n' +
        '  --auth-provider-arg=client-secret="' +
        this.kubecfg.clientSecret +
        '" \\\n' +
        '  --auth-provider-arg=id-token="' +
        this.kubecfg.idToken +
        '" \\\n' +
        '  --auth-provider-arg=refresh-token="' +
        this.kubecfg.refreshToken +
        '" \\\n' +
        '  --auth-provider-arg=idp-issuer-url="' +
        this.kubecfg.issuer +
        '"\n\n' +
        "# Associate your user with an existing cluster\n" +
        'kubectl config set-context cluster.' + window.location.hostname.split('.').slice(-2).join('.') + ' --cluster cluster.' + window.location.hostname.split('.').slice(-2).join('.') + ' --user="' +
        'user@' + window.location.hostname.split('.').slice(-2).join('.') +
        '"'
      );
    }
  },
  created: function() {
    var q = decodeURI(location.search.substr(1))
      .replace(/"/g, '\\"')
      .replace(/&/g, '","')
      .replace(/=/g, '":"');
    var query = "";
    if (q != "") {
      query = JSON.parse('{"' + q + '"}');
    }
    var url = "kubecfg?" + $.param(query);

    var _this = this;
    this.axios
      .get(url)
      .then(function(response) {
        _this.kubecfg = response.data;
        if(_this.kubecfg.email == "") {
          _this.kubecfg.email = "kuberos";
        }
      })
      .catch(function(error) {
        _this.error = error;
      });
  }
};
</script>

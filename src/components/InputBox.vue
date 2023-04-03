<template>
  <div>
    <input type="textarea" v-model="query" @keyup.enter="analyzeQuery" />
    <div id="results" v-if="action || type || filter">
      <p>Action: {{ action }}</p>
      <p>Type: {{ type }}</p>
      <p>Filter: {{ filter }}</p>
      <p>Url: {{ miaUrl }}</p>
      <button @click="goToMia"><i class="fas fa-external-link-alt"></i>Open link</button>
    </div>
  </div>
</template>

<script>

export default {
  data() {
    return {
      query: "",
      action: "",
      type: "",
      filter: "",
      miaUrl: ""
    };
  },
  methods: {
    async analyzeQuery() {
      const types = ['content', 'component', 'particle', 'placement', 'program', 'product-class', 'assembly']
      const prompts = [
        { role: 'user', content: `You are a Helpbot for an internal application. Users will ask you questions to help them with something. You should breakdown their query and respond with a json object with keys: action, type, filter. Actions can be "view", "create", or "edit". Type can be one from the following list: ${types}` },
        { role: 'assistant', content: 'Okay. I will do that.' },
        { role: 'user', content: 'I want to create a new content record' },
        { role: 'assistant', content: '{"action": "create", "type": "content", "filter": {}}' },
        { role: 'user', content: 'View all assemblies' },
        { role: 'assistant', content: '{"action": "view", "type": "assembly", "filter": {"list_all": true}}' },
        { role: 'user', content: 'I want to change the metadata of particle with id X18273' },
        { role: 'assistant', content: '{"action": "edit",  "type": "particle",  "filter": {"id": "X18273"}}' },
        { role: 'user', content: this.query }
      ]
      const response = await fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${process.env.VUE_APP_OPENAI_API_KEY}`,
        },
        body: JSON.stringify({
          model: 'gpt-3.5-turbo',
          messages: prompts,
        }),
      });
      const data = await response.json();
      const choices = data?.choices[0];
      const content = JSON.parse(choices.message.content)
      const { action, type, filter } = content;
      this.action = action;
      this.type = type;
      this.filter = filter;
      this.generateUrl({ action, filter, type })
    },
    goToMia() {
      const url = this.miaUrl;
      window.open(url, "_blank");
    },
    generateUrl(data) {
      let { action: a, type: t, filter: f } = data
      const root = 'https://mia-staging.benchmarkconnect.com'
      let url = ''
      if (a === 'edit' && t === 'assembly') {
        url = `${root}/constituents-management`
        let assemblyCode = (f['id'] || '').startsWith('A-') ? f['id'] : `A-${f['id']}`
        url = `${url}/${assemblyCode}`
      } else {
        url = `${root}/editors/${t}`
        let filters = this.generateFilters(f, t);
        url = `${url}?q=${filters}`
      }
      this.miaUrl = url
    },
    generateFilters(filters, type) {
      let options = {}
      if (filters.list_all) {
        options = {
          [`${type}_code`]: { values: null, exactMatch: true, notMatch: true }
        }
      }
      return encodeURIComponent(JSON.stringify(options)).replaceAll('%', '%25')
    },
  },
};
</script>

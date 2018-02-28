<template>
    <div>

    <section class="section">
      <div class="container">
        <h1 class="title">Welcome to the Bounty Board!</h1>
        <h2 class="subtitle">
          Your one stop to find your dream job.
        </h2>
        <p>If this page doesn't make sense to you, check out the <router-link to="/about">About</router-link> page.
        </p>
      </div>
    </section>

    <section class="section">
      <div class="container">
        <h2 class="title">Search for Jobs</h2>
        <div class="field">
          <label class="label">Keywords</label>
          <div class="control">
            <input class="input" type="text" id="keywords" placeholder="Job Title, Description" v-model="searchField">
          </div>
        </div>
        <div class="field">
          <label class="label">Location</label>
          <div class="control">
            <input class="input" type="text" id="location" placeholder="City, State" v-model="locationField">
          </div>
        </div>
        <div class="field">
          <div class="control">
            <button class="button is-success is-small" id="customize-button" v-on:click="toggleModal()">Customize Job Boards</button>
          </div>
        </div>
        <div class="field">
          <div class="control">
            <button class="button is-link is-large" id="search-jobs" v-on:click="searchJobs()">Search Jobs</button>
          </div>
        </div>
      </div>
    </section>

    <section class="section">
      <div class="container" id="searchResults">
        <div class="box" v-for="post in results">
          <article class="media">
            <div class="media-content">
              <p><strong>{{ post.title }}</strong><br>
              Organization: {{ post.organization }} <br>
              Location: {{ post.location }} <br>
              <a :href="post.link">Link to Post</a><br>
              Source: {{ post.source }} </p>
            </div>
          </article>
        </div>
      </div>
      <div class="container" v-if="loading">
        <h2 class="title">Loading...</h2>
      </div>
    </section>

    <div class="modal" id="board-modal">
      <div class="modal-background"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">Select Job Boards</p>
          <button class="delete" aria-label="close" id="close-modal" v-on:click="toggleModal()"></button>
        </header>
        <section class="modal-card-body">
            <div class="control">
              <label class="checkbox">
                <input type="checkbox" id="adzuna" v-model="searchAdzuna">Adzuna
              </label>
            </div>
            <div class="control">
              <label class="checkbox">
                <input type="checkbox" id="search" v-model="searchSearch">Search.gov Jobs
              </label>
            </div>
        </section>
        <footer class="modal-card-foot">
          <button class="button is-success" id="save-boards" v-on:click="toggleModal()">Save changes</button>
        </footer>
      </div>
    </div>

    </div>
</template>

<script>
 export default {
     name: 'Home',
     data () {
       return {
         searchField: '',
         locationField: '',
         searchAdzuna: true,
         searchSearch: true,
         results: [],
         loading: false
       }
     },
     methods: {
       toggleModal: function(){
         var modal = document.getElementById("board-modal");
         modal.classList.toggle("is-active");
       },
       linkStrings: function(divider) {
         return function(a,b){ return a.concat(divider + b); };
       },
       stripHtml: function(string) {
         var tmp = document.createElement("div");
         tmp.innerHTML = string;
         return tmp.textContext || tmp.innerText || "";
       },
       handleError: function(error) {
         console.log('Error: ' + JSON.stringify(error,null,2));
       },
       searchJobs: function() {
         var keywords = this.searchField.trim().split(" ");
         var location = this.locationField.trim().split(" ");
         this.results = [];
         this.loading = true;
         if(this.searchAdzuna){
           this.callAdzuna(keywords, location);
         }
         if(this.searchSearch){
           this.callSearch(keywords, location);
         }
       },
       callAdzuna: function(keywords, location) {
         var appId = '8a3d5455';
         var appKey = '42b0a8c582284eb8e69d793198ec1c39';
         var numResults = '20';
         var url = 'http://api.adzuna.com/v1/api/jobs/us/search/1?app_id=' + appId + '&app_key=' + appKey + '&results_per_page=' + numResults + '&what=' + keywords.reduce(this.linkStrings('%20')) + '&where=' + location.reduce(this.linkStrings('%20')) + '&content-type=application/json';
         this.$http.get(url).then(function(json){
             this.loading = false;
             for(var i = 0; i < json.body.results.length; ++i){
               this.results.push({ 
                 title: this.stripHtml(json.body.results[i].title),
                 organization: json.body.results[i].company.display_name,
                 location: json.body.results[i].location.display_name,
                 link: json.body.results[i].redirect_url,
                 source: 'Adzuna'
               });
             }
         }, this.handleError );
       },
       callSearch: function(keywords, location) {
         var url = 'https://jobs.search.gov/jobs/search.json?query=' + keywords.reduce(this.linkStrings('+'));
         if(location.length > 0){
           url += '+in+' + location.reduce(this.linkStrings('+'));
         }
         this.$http.get(url).then(function(json){
           this.loading = false;
           for(var i = 0; i < json.body.length; ++i){
             this.results.push({ 
               title: json.body[i].position_title,
               organization: json.body[i].organization_name,
               location: json.body[i].locations[0],
               link: json.body[i].url,
               source: 'Search.gov'
             });
           }
         }, this.handleError );
       }
     }
 }
</script>

<style scoped>

</style>

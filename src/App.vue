<template>
  <div id="app">
    <div class="container is-fluid">      
    <div class="columns">
      <div class="column is-one-fifth">
        <article class="panel is-link">
          <div id="header">
            <div class="panel-heading" slot="trigger">
              Reading A-Z
                <b-dropdown
                    :scrollable="isScrollable"
                    :max-height="maxHeight"
                    v-model="currentMenu"
                    aria-role="list"
                    :triggers="['hover']"
                    class="is-success"
                >
                <button class="button has-background-light" slot="trigger">
                    <span>Level {{currentMenu}}</span>
                    <b-icon icon="menu-down"></b-icon>
                </button>

                    <b-dropdown-item 
                        v-for="(menu, index) in menus"
                        :key="index"
                        :value="menu" aria-role="listitem">
                        <div class="media">
                            <div class="media-content">
                                <h3>Level {{menu}}</h3>
                            </div>
                        </div>
                    </b-dropdown-item>
                </b-dropdown>

            </div>
            <div class="panel-block">
              <p class="control has-icons-left">
                <input class="input is-primary" v-model="keyword" type="text" placeholder="Search">
                <span class="icon is-left">
                  <i class="fas fa-search" aria-hidden="true"></i>
                </span>
                <span class="icon is-right" v-show="keyword"  @click="clearInput" >
                  <i class="far fa-times-circle" aria-hidden="true"></i>
                </span>
              </p>
            </div>
          </div>

          <div id="booklist">
            <a @click="setId(book.id)" v-for="book in currentBooks" :key="book.id" class="panel-block" :class="book.id==currentId?'active':''" >
              <span class="panel-icon">
                <i class="fas fa-book" aria-hidden="true"></i>
              </span>
              {{book.title}}
            </a>
          </div>
        </article>
      </div>
      <div class="column">
        <loading :active.sync="isLoading" 
        :can-cancel="true"
        :is-full-page="true"></loading>
        <flipbook class="flipbook" 
          :pages="pages" 
          :singlePage="true"
          :centering="false"
          ref="flipbook"
          :zooms=[1,2]
          :key="componentKey"
          >
        </flipbook>
      </div>

      <div class="column is-one-fifth panel">

          <div class="info">
            <h3>{{detail.title}}</h3>
            <p>{{detail.subtle}}, {{currentId}}</p>
          </div>

          <div class="info" v-for="(title, index) in metaTitles" :key=index>
            <h3>{{title}}</h3>
            <p v-html="detail.meta[index]"></p>
          </div>
          <div class="info">
            <h3>Lesson Resources</h3>
            <p><a :href="detail.lesson" target="_blank">Guided Reading Lesson</a></p>
            <p><a :href="detail.worksheet" target="_blank">All Worksheets</a></p>
            <p v-html="detail.description"></p>
          </div>
      </div>
    </div>
    
    </div>
  </div>
</template>

<script>
// import Raz from './components/Raz.vue'
import raz from "./assets/raz.json";
import pageData from "./assets/pages.json";
import Flipbook from 'flipbook-vue';

// Import component
import Loading from 'vue-loading-overlay';
// Import stylesheet
import 'vue-loading-overlay/dist/vue-loading.css';

export default {
  name: 'App',
  components:{
    Flipbook,
    Loading
  },
  data() {
    return {
        isLoading: false,
        raz,
        pageData,
        componentKey: 0,
        isScrollable: true,
        maxHeight: 500,
        currentMenu: "aa",
        keyword:"",
        currentId: 2917,
        menus: ["aa","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","Z1","Z2"],
    }
  },
  mounted(){
    // this.axios.get('url')
    //   .then((response) => {
    //     console.log(response.data)
    //   })
    window.addEventListener("keydown", (i) => {
        const e = this.$refs.flipbook
        if (e){
            return 37 === i.keyCode && e.canFlipLeft && e.flipLeft(),
            39 === i.keyCode && e.canFlipRight ? e.flipRight() : void 0,
            13 === i.keyCode && e.canFlipRight ? e.flipRight() : void 0, //回车
            32 === i.keyCode && e.canFlipRight ? e.flipRight() : void 0 //空格
        }
      }
    )
  },
  computed:{
    metaTitles(){
      let titles = ["High-Frequency Words","Story Words","Reading Strategy","Comprehension","Phonological Awareness","Phonics","Grammar and Mechanics","Word Work"];
      console.log(this.detail.meta.length )
      if(this.detail.meta.length === 7){
        titles.shift();
      }
      return titles;
    }
    ,
    pages(){
      const id = this.currentId
      const meta = this.pageData[id];
      let pages = []
      for (let index = 0; index < meta.count; index++) {
        const i = index + meta.from;
        pages.push('https://mi.content.readinga-z.com/raz_book_image/'+id+'/projectable/large/1/book/page-'+i+'.jpg')
      }
      return pages
    },
    currentBooks(){
      return this.raz.filter((item)=>{
        if(this.keyword){
          var re = new RegExp(this.keyword, 'i');
          return item.level == this.currentMenu && item.title.match(re)
        }else{
          return item.level == this.currentMenu
        }
        
      })
    },
    currentLevelCount(){
      return this.currentBooks.length
    },
    detail(){
      let detail = require("./assets/jsons/"+this.currentId+".json");
      detail.worksheet = "https://cf.content.readinga-z.com/pdfsite/"+this.currentId+"/"+detail.pdf+"_wksh.pdf";
      detail.lesson = "https://cf.content.readinga-z.com/pdfsite/"+this.currentId+"/"+detail.pdf+"_lblp.pdf";
      return detail
    }
  },
  methods:{
    clearInput(){
      this.keyword = ''
    },
    setId(id){
      if(this.currentId !== id){
        this.currentId = id
        this.isLoading = true
        this.forceRerender()
        setTimeout(() => {
          this.isLoading = false
        },500)
      }
    },
    forceRerender() {
      this.componentKey += 1;
    }
  }
}
</script>

<style>
.flipbook {
  height: 95vh;
}
.column{
  max-height: 100vh;
  overflow-y: scroll;
}
#app .container.is-fluid {
  padding: 0;
}
#booklist {
  max-height: 95%;
  overflow-y: scroll;
  padding-top: 125px;
}
#header{
  position: fixed;
  background: white;
}
.info{
  border-bottom: 1px solid #666;
  padding: .5em;
}
.info h3{
  font-weight: bold;
}
.is-right{
  right: 0;
}
.control.has-icons-left .icon.is-right{
      pointer-events: auto;
}
#booklist .active{
  background: #ededed;
}
</style>

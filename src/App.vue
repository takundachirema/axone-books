<template>
  <div id="app">
    <notifications group="message" />
    <sidebar-menu
      :menu="menu"
      :show-one-child="true"
      :collapsed="true"
      :hideToggle="true"
      @item-click="onItemClick">
      <div slot="header" class="top-menu-item">
        <img src="./assets/images/book_shelf.png">
      </div>
    </sidebar-menu>

    <router-view :searchQuery="searchText"/>
    
    <book-popup 
      v-if="bookPopupIsVisible" 
      @close="closeBookPopup" 
      :book="book" 
      :pk="pk"
      :node="node"
      >
    </book-popup>

    <header v-show="show_header" class="header">
      <div class="header__search row">
        <div class="input-field col s1">
          <i class="material-icons prefix">search</i>
          <input id="icon_prefix header__search-input" type="text" v-model.trim="searchQuery" @keyup.enter="search">
          <label for="icon_prefix">search for book...</label>
        </div>
      </div>
    </header>
  </div>
</template>

<script>
  import M from 'materialize-css'
  import storage from './storage.js'
  import BookPopup from './components/BookPopup.vue'

  export default {
    name: 'App',
    components: {BookPopup},
    methods: {
      search() {
        this.searchText = this.searchQuery
      },
      storeBooks(books){
        for (var i=0;i<books.length;i++){
          console.log(JSON.stringify(books[i]));
          localStorage.setItem(books[i].id,JSON.stringify(books[i]))
        }
      },
      openBookPopup(book, pk, node) {
        this.bookPopupIsVisible = true;
        this.book = book;
        this.pk = pk;
        this.node=node;
        document.querySelector('body').classList.add('hidden');
      },
      closeBookPopup(){
        storage.createBookPopup = false;
        this.bookPopupIsVisible = false;
        document.querySelector('body').classList.remove('hidden');
        //window.history.back();
      },
      onToggleCollapse (collapsed) {
        this.collapsed = collapsed
      },
      onItemClick (event, item) {
        console.log('onItemClick')
        console.log(event)
        console.log(item)
      },
      filter: function() {
        console.log("add name")
      },
      // Router After Leave
      afterLeave(){
        document.querySelector('body').scrollTop = 0;
      },
      // Detect if touch device
      isTouchDevice() {
        return 'ontouchstart' in document.documentElement;
      },
      showHeader(show) {
        this.show_header = show;
      },
      hideMenu(number, hide){
        this.menu[number].hidden = hide;
      },
      selectMenu(number, url){
        this.menu[number].hidden = false;
        this.menu[number].child.push({href: url});
      },
      /**
       * type: error, warn, success
       */
      showMessage(type, title, message, duration=4000){
        this.$notify({
          group: 'message',
          type: type,
          title: title,
          duration: duration,
          text: message
        });
      }
    },
    mounted () {
      M.AutoInit()
    },
    created () {
      this.collapsed = true;
      window.addEventListener('popstate', this.onHistoryState);
      window.addEventListener('pagehide', this.changeHistoryState);
      eventHub.$on('openBookPopup', this.openBookPopup);
      eventHub.$on('closeBookPopup', this.closeBookPopup);
      eventHub.$on('setSearchQuery', this.setSearchQuery);
      eventHub.$on('requestToken', this.requestToken);
      eventHub.$on('setUserStatus', this.setUserStatus);
      eventHub.$on('showHeader', this.showHeader);
      eventHub.$on('hideMenu', this.hideMenu);
      eventHub.$on('selectMenu', this.selectMenu);
      eventHub.$on('storeBooks', this.storeBooks);
      eventHub.$on('showMessage', this.showMessage);
      if (this.isTouchDevice()) {
        document.querySelector('body').classList.add('touch');
      }
    },
    data() {
      return {
        bookPopupIsVisible: false,
        bookPopupHistoryVisible: false,
        book: {},
        pk: '',
        searchQuery: '',
        searchText: '',
        loading: true,
        show_header: false,
        menu: [
          {
            header: true,
            title: 'Axone Books',
            hiddenOnCollapse: true
          },
          {
            href: '/',
            title: 'Library',
            icon: 'fa fa-book'
          },
          {
            href: '/read',
            title: 'Read',
            hidden: true,
            icon: 'fa fa-book-reader',
            class: 'vsm--link_active',
            child: []
          },
          {
            href: '/explore',
            title: 'Explore',
            hidden: true,
            icon: 'fa fa-cogs',
            class: 'vsm--link_active',
            child: []
          },
          {
            href: '/publish',
            title: 'Publish',
            icon: 'fa fa-globe'
          },
          {
            href: '/info',
            title: 'Information',
            icon: 'fa fa-info'
          }
        ]
      }
    }
  }
</script>

<style lang="scss">
  @import './assets/css/variables';
  @import './assets/css/media-queries';
  @import './assets/css/app';
  @import './assets/css/templatemo-style';
</style>
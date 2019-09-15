<template>
  <div id="app">
    <div class="nav" :class="{'moon' : mode === 'moon'}">
      <h1 class="logo">FreeNote<i>.</i></h1>
      <button class="addpen" @click="addMemo"><i></i>新增筆記</button>
      <ul>
        <li class="allpen" @click="onlyImportant = false"><i></i>所有筆記</li>
        <li class="path" @click="onlyImportant = true"><i></i>捷徑</li>
        <li class="tag"><i></i>標籤</li>
        <li class="calendar"><i></i>月曆</li>
        <li class="share"><i></i>與我共用</li>
        <li class="bucket"><i></i>垃圾桶</li>
        <li class="mode">模式
          <i class="sun" @click="mode = 'sun'"></i>
          <i class="moon" @click="mode = 'moon'"></i>
        </li>
        <li class="account">
          <i class="accountIcon"></i>
          Michelle
          <i class="more"></i>
        </li>
      </ul>
    </div>
    <div class="list" @click="edit(selected)" :class="{'moon' : mode === 'moon'}">
      <label for="search"></label>
      <input type="text" id="search" placeholder="搜尋您的筆記">
      <h2>{{onlyImportant === false ? "所有筆記" : "捷徑"}}</h2>
      <i class="more"></i>
      <i class="display" @click="itemList.isOpen = !itemList.isOpen"
      :class="{'block' : itemList.display === 'block',
      'onlyTitle' : itemList.display === 'onlyTitle'}"></i>
      <ul class="displayBox" v-if="itemList.isOpen">
        <li @click="changeDisplay('card')"><i class="card"></i>卡片檢視</li>
        <li @click="changeDisplay('block')"><i class="block"></i>摘要檢視</li>
        <li @click="changeDisplay('onlyTitle')"><i class="onlyTitle"></i>文字列表檢視</li>
      </ul>
      <ul class="itemList" :class="{'block' : itemList.display === 'block',
        'onlyTitle' : itemList.display === 'onlyTitle'}">
        <li v-for="(item, key) in memoFilter" :key="key" :class="{'selected' : key === selected}"
        @click.stop="edit(key)">
          <i class="angle" @click.stop="additional = additional === key ? null : key"></i>
          <ul class="additionalBox" v-if="key === additional">
            <li @click.stop="addStar(key, item.id)"><i class="importantIcon"></i>標記</li>
            <li @click.stop="removeMemo(item.id)"><i class="delete"></i>刪除</li>
          </ul>
          <h3 class="title">
            <i class="star" v-if="item.isImportant"></i>
            {{item.text | title | replaceHTML }}
          </h3>
          <p class="text">{{item.text | text | replaceHTML}}</p>
          <h4 class="tag" v-if="item.tag.length !== 0">{{item.tag[0]}}</h4>
          <span class="date">
            <i class="attachment" v-if="item.text.includes('href')"></i>
            {{item.date | timeDisplay}}
          </span>
        </li>
      </ul>
    </div>
    <div class="content" :class="{'moon' : mode === 'moon'}">
      <quill-editor
        v-model="content"
        ref="myQuillEditor"
        :options="editorOption"
        @blur="onEditorBlur($event)">
      </quill-editor>
      <input type="text" placeholder="Add + " v-model="newTag"
      @keypress.stop.prevent.13="addTag">
      <ul class="tagWrap" v-if="selected !== null && memo[selected]">
        <li v-for="(item, key) in memo[selected].tag" :key="key">
          <button>{{item}}
            <i @click="removeTag(key)">x</i>
          </button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
const db = window.firebase.firestore();

export default {
  name: 'home',
  data() {
    return {
      // 富文本
      content: '',
      editorOption: {
        modules: {
          toolbar: [
            ['bold', 'italic', 'underline'],
            [{ indent: '-1' }, { indent: '+1' }],
            [{ align: [] }],
            [{ list: 'bullet' }],
            [{ color: [] }, { background: [] }],
            [{ font: [] }],
            [{ size: ['small', false, 'large', 'huge'] }],
            ['clean'],
            ['link', 'image', 'video'],
          ],
        },
        placeholder: '請寫下內容',
        theme: 'snow',
      },
      // firebase 檔案
      memo: [],
      // 選取編輯
      selected: null,
      // 右上選單 => 加星號，刪除
      additional: null,
      // 筆記顯示方式
      itemList: {
        isOpen: false,
        display: 'card',
      },
      onlyImportant: false,
      mode: 'sun',
      // 新增標籤
      newTag: '',
    };
  },
  firestore: {
    memo: db.collection('NOTE'),
  },
  methods: {
    edit(key) {
      this.selected = this.selected === key ? null : key;
      this.content = this.selected !== null ? this.memo[this.selected].text : '';
    },
    convertor(value) {
      if (value < 10) {
        return `memo00${String(value)}`;
      } else if (value < 100) {
        return `memo0${String(value)}`;
      }
      return `memo${String(value)}`;
    },
    addMemo() {
      const id = this.memo.length !== 0 ?
        this.convertor(Number(this.memo[this.memo.length - 1].id.substr(4, 3)) + 1) : 'memo001';
      db.collection('NOTE').doc(id).set({
        text: '',
        isImportant: false,
        date: Number(new Date()),
        tag: [],
      }).then(() => {
        this.edit(this.memo.length - 1);
      });
    },
    onEditorBlur() {
      // 離開對焦
      if (this.selected === null) { return; }
      db.collection('NOTE').doc(this.memo[this.selected].id).update({
        text: this.content,
      });
    },
    changeDisplay(value) {
      this.itemList.display = value;
      this.itemList.isOpen = false;
    },
    addTag() {
      if (this.selected === null) { return; }
      if (this.newTag.length > 7) {
        alert('字數太長啦！');
        this.newTag = '';
        return;
      }
      const newArr = [...this.memo[this.selected].tag];
      newArr.push(this.newTag);
      db.collection('NOTE').doc(this.memo[this.selected].id).update({
        tag: newArr,
      }).then(() => {
        this.newTag = '';
      });
    },
    removeTag(key) {
      const newArr = [...this.memo[this.selected].tag];
      newArr.splice(key, 1);
      db.collection('NOTE').doc(this.memo[this.selected].id).update({
        tag: newArr,
      });
    },
    addStar(key, id) {
      db.collection('NOTE').doc(id).update({
        isImportant: !this.memo[key].isImportant,
      }).then(() => {
        this.additional = null;
      });
    },
    removeMemo(id) {
      db.collection('NOTE').doc(id).delete().then(() => {
        this.additional = null;
      });
    },
  },
  filters: {
    title(value) {
      return String(value.split('<br>', 1));
    },
    text(value) {
      const newArr = value.split('<br>');
      newArr.splice(0, 1);
      return String(newArr);
    },
    replaceHTML(str) {
      return str.replace(/<[^>]+>/g, '').replace(/&nbsp;/ig, '');
    },
    timeDisplay(value) {
      const dt = new Date(value);
      return `${dt.getFullYear()}/${dt.getMonth() + 1}/${dt.getDate()}`;
    },
  },
  computed: {
    memoFilter() {
      if (this.onlyImportant === true) {
        return this.memo.slice().filter(item => item.isImportant === true);
      }
      return this.memo;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
@import '../sass/home.scss';
</style>

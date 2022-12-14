<template>
  <form>
    <h1>Lightning Spell - NovelAI spell generator</h1>
    <div class="data">
      <ul class="cat-list">
        <li v-for="(category, categoryIndex) in categories" :key="categoryIndex" class="cat-container">
          <input class="cat-name" type="text" v-model="category.name" placeholder="category name">
          <ul class="word-list">
            <li v-for="(word, wordIndex) in category.words" :key="wordIndex" class="word-container" :class="{'disabled': word.disabled, 'editing': word.editing}">
              <button class="disable-word-btn material-icons" type="button" @click="toggleDisabled(word)"></button>
              <div class="edit" v-if="word.editing">
                <input class="word" type="text" v-model="word.name" @input="updateSpell" placeholder="word">
                <input class="word-note" type="text" v-model="word.note" @input="updateSpell" placeholder="word note">
                <input class="word-priority" type="number" v-model="word.priority" @change="updateSpell" @input="updateSpell" min="-9" max="10">
                <span class="word-priority-label material-icons"></span>
              </div>
              <div class="text" v-if="!word.editing">
                <div class="word-text">{{word.name}}</div>
                <div class="word-note-text">{{word.note}}</div>
                <div class="word-priority-text" :data-priority="word.priority"></div>
              </div>
              <button class="edit-word-btn material-icons" type="button" @click="toggleEditing(word)"></button>
              <button class="delete-word-btn material-icons" type="button" @click="removeWord(categoryIndex, wordIndex)"></button>
            </li>
          </ul>
          <button class="add-word-btn material-icons" type="button" @click="addWord(categoryIndex)"></button>
          <button class="delete-cat-btn material-icons" type="button" @click="removeCategory(categoryIndex)"></button>
        </li>
      </ul>
      <button class="add-cat-btn material-icons" type="button" @click="addCategory"></button>
    </div>
    <div class="import">
      <h2>JSON Import</h2>
      <input id="import" type="file" @change="dataImport" v-if="file">
    </div>
    <div class="export">
      <h2>JSON Export</h2>
      <button id="export" type="button" @click="dataExport">JSON Export</button>
    </div>
    <div class="reset">
      <h2>Etc</h2>
      <button id="reset-btn" type="button" @click="resetData">All reset</button>
    </div>
    <div class="result">
      <input id="spell" type="text" v-model="spell" placeholder="spell"  @click="copySpell" readonly>
    </div>
  </form>
</template>

<script>

export default {
  name: 'App',

  data() {
    return {
      categories: [{
        "name": "cat name",
        "words": [{
          "name": "word",
          "note": "word note",
          "priority": 0,
          "disabled": false,
          "editing": true
        }]
      }],
      spell: '',
      file: true
    }
  },
  mounted() {
    this.$nextTick(function(){
      if(localStorage.getItem('categories')){
        this.categories = JSON.parse(localStorage.getItem('categories'))
      }
      this.updateSpell()
    })
  },
  methods: {
    addCategory() {
      this.categories.push({
        "name": "cat name",
        "words": [{
          "name": "word",
          "note": "word note",
          "priority": 0,
          "disabled": false,
          "editing": true
        }]
      })
      this.saveCategory()
    },
    removeCategory(index) {
      this.categories.splice(index, 1)
      this.saveCategory()
    },
    addWord(categoryIndexOfAddWord) {
      this.categories.forEach(function(category, categoryIndex){
        console.log(categoryIndex +'and'+ categoryIndexOfAddWord)
        if(categoryIndex==categoryIndexOfAddWord){
          category.words.push({
            "name": "word",
            "note": "word note",
            "priority": 0,
            "disabled": null,
            "editing": true
          })
        }
      })
      this.saveCategory()
    },
    removeWord(categoryIndexOfRemoveWord, wordIndexOfRemoveWord) {
      this.categories.forEach(function(category, categoryIndex){
        if(categoryIndex==categoryIndexOfRemoveWord){
          category.words.splice(wordIndexOfRemoveWord, 1)
        }
      })
      this.saveCategory()
    },
    saveCategory () {
      localStorage.categories = JSON.stringify(this.categories)
      this.updateSpell()
    },
    updateSpell() {
      var spell = '', spellWordBefore = '', spellWordAfter = '', i=0;
      this.categories.forEach(function(category){
        category.words.forEach(function(word){
          if(!word.disabled){
            spellWordBefore = '', spellWordAfter = ''
            if(word.priority > -10 && word.priority < 0){
              for(i=0; i < -(word.priority); i++){
                spellWordBefore += '['
                spellWordAfter += ']'
              }
            }else if(word.priority > 0 && word.priority < 11){
              for(i=0; i < word.priority; i++){
                spellWordBefore += '{'
                spellWordAfter += '}'
              }
            }
            spell += spellWordBefore + word.name + spellWordAfter + ', '
          }
        })
      })
      this.spell = spell
    },
    resetData() {
      this.categories = [{
        "name": "cat name",
        "words": [{
          "name": "word",
          "note": "word note",
          "priority": 0,
          "disabled": false,
          "editing": true
        }]
      }]
      this.updateSpell()
      localStorage.removeItem('categories')
    },
    dataImport(e) {
      var file = e.target.files[0]
      var errors = []
      if (file.size > 1000000) {
        errors.push('ファイルサイズを1MB以下にしてください。')
      }
      if(file.type!="application/json"){
        errors.push('ファイル形式をjsonにしてください。')
      }
      if(!errors.length){
        var vue = this
        var reader = new FileReader()
        reader.onload = function(e){
          vue.categories = JSON.parse(e.target.result)
          vue.saveCategory()
        }
        reader.readAsText(file)
      }
      this.file = false
      this.$nextTick(function () {
        this.file = true
      })
    },
    dataExport() {
      var spellDataString = JSON.stringify(this.categories)
      var blob = new Blob([spellDataString], {type:'application/json'})
      var dataUrl = URL.createObjectURL(blob)
      var downloadLink = document.createElement('a');
      downloadLink.href = dataUrl;
      downloadLink.download = `lightningspell.json`
      document.body.appendChild(downloadLink)
      downloadLink.click()
      downloadLink.parentNode.removeChild(downloadLink)
    },
    toggleDisabled(word) {
      word.disabled = !word.disabled
      this.saveCategory()
    },
    toggleEditing(word) {
      word.editing = !word.editing
      this.saveCategory()
    },
    copySpell() {
      navigator.clipboard.writeText(this.spell)
    }
  }
}
</script>

<style lang="sass">
.v-enter-active,
.v-leave-active
  transition: width .2s;
@keyframes bounce-in
  0%
    max-width: 0
  100%
    max-width: 100%
html
  font-size: 10px
body
  background: #333
  color: #fff
h1
  font-size: 3.2rem
  line-height: 1
  margin-bottom: 2rem
form
  padding: 2rem 2rem 7rem
.data
  margin-bottom: 2rem
  .option
    margin-bottom: 2rem
    label
      display: flex
      align-items: center
      font-size: 1.2rem
      input
        margin-right: 0.5em
  .cat-list
    margin-bottom: 1rem
    > li
      background: #666
      border-radius: 1rem
      padding: 0 1rem
      margin-bottom: 1rem
      position: relative
      transition: all .2s
      &:last-child
        margin-bottom: 0
      .cat-name
        margin-bottom: 1rem
        border: 0
        border-bottom: 1px solid #fff
        width: 100%
        background-color: transparent
        font-weight: bold
        line-height: 1
        padding: 0
        display: block
        width: 100%
        font-size: 2rem
        height: 4rem
      .add-word-btn
        border: 0
        background-color: #fff
        color: #666
        margin-bottom: 1rem
        font-size: 1.6rem
        padding: 0.2rem
        width: 2rem
        height: 2rem
        border-radius: 1.5rem
        &::before
          content: 'add'
      .delete-cat-btn
        border: 0
        background-color: #fff
        color: #666
        margin-bottom: 1rem
        font-size: 1.6rem
        padding: 0.2rem
        width: 2rem
        height: 2rem
        border-radius: 1.5rem
        position: absolute
        top: -0.5rem
        right: -0.5rem
        &.delete-cat-btn::before
          content: 'close'
      .word-list
        display: flex
        flex-wrap: wrap
        white-space: nowrap
        > li
          border-radius: 10px
          overflow: hidden
          display: flex
          margin: 0 10px 10px 0
          transition: all .2s
          padding-right: 3rem
          position: relative
          .text
            position: relative
            .word-text
              font-size: 1.6rem
              line-height: 1
              padding: 1rem
              height: 3.6rem
              background-color: #fff
              color: #333
              font-weight: bold
            .word-note-text
              font-size: 1.2rem
              line-height: 1
              padding: .5rem 1rem .5rem 2.8rem
              height: 2.2rem
              background-color: #eee
              color: #666
            .word-priority-text
              font-size: 1rem
              line-height: 1
              padding: .3rem 0
              width: 1.8rem
              height: 1.8rem
              border-radius: 0.9rem
              text-align: center
              background-color: #fff
              color: #333
              position: absolute
              left: 0.5rem
              bottom: 0.2rem
              &::before
                content: "0"
              &[data-priority="-9"]
                &::before
                  content: "-9"
              &[data-priority="-8"]
                &::before
                  content: "-8"
              &[data-priority="-7"]
                &::before
                  content: "-7"
              &[data-priority="-6"]
                &::before
                  content: "-6"
              &[data-priority="-5"]
                &::before
                  content: "-5"
              &[data-priority="-4"]
                &::before
                  content: "-4"
              &[data-priority="-3"]
                &::before
                  content: "-3"
              &[data-priority="-2"]
                &::before
                  content: "-2"
              &[data-priority="-1"]
                &::before
                  content: "-1"
              &[data-priority="0"]
                &::before
                  content: "0"
              &[data-priority="1"]
                &::before
                  content: "1"
              &[data-priority="2"]
                &::before
                  content: "2"
              &[data-priority="3"]
                &::before
                  content: "3"
              &[data-priority="4"]
                &::before
                  content: "4"
              &[data-priority="5"]
                &::before
                  content: "5"
              &[data-priority="6"]
                &::before
                  content: "6"
              &[data-priority="7"]
                &::before
                  content: "7"
              &[data-priority="8"]
                &::before
                  content: "8"
              &[data-priority="9"]
                &::before
                  content: "9"
              &[data-priority="10"]
                &::before
                  content: "10"
          .edit
            background-color: #9c9
            position: relative
            width: 22rem
            input
              display: block
              border: 0
            .word
              font-size: 1.6rem
              line-height: 1
              padding: 0.5rem
              width: 20rem
              height: 2.6rem
              border-radius: 0.5rem
              background-color: #fff
              color: #333
              font-weight: bold
              margin: 0.5rem
            .word-note
              font-size: 1.2rem
              line-height: 1
              padding: .25rem 0.5rem
              width: 13.5rem
              height: 1.7rem
              border-radius: 0.5rem
              background-color: #eee
              color: #666
              margin: 0 0 0.5rem 7rem
            .word-priority-label
              font-size: 15px
              background-color: #999
              border-radius: 0.5rem 0 0 0.5rem 
              width: 2rem
              height: 1.7rem
              text-align: center
              position: absolute
              left: 0.5rem
              bottom: 0.5rem
              &::before
                content: 'priority_high'
            .word-priority
              font-size: 1.2rem
              line-height: 1
              padding: .25rem 0.5rem
              width: 4rem
              height: 1.7rem
              border-radius: 0 0.5rem 0.5rem 0 
              background-color: #eee
              color: #666
              position: absolute
              left: 2.5rem
              bottom: 0.5rem
          .disable-word-btn
            display: block
            width: 3rem
            height: 100%
            border: 0
            background-color: #333
            &:focus-visible
              border: 1px dotted #fff
            &::before
              content: 'visibility'
            &:hover::before
              content: 'visibility_off'
          .edit-word-btn
            display: block
            width: 3rem
            height: 2.9rem
            border: 0
            background-color: #999
            position: absolute
            top: 0
            right: 0
            transition: all .2s
            &:hover
              background-color: #393
            &:focus-visible
              border: 1px dotted #fff
            &::before
              content: 'edit'
          &.editing
            .edit-word-btn
              background-color: #393
              &::before
                content: 'done'
          .delete-word-btn
            display: block
            width: 3rem
            height: 2.9rem
            border: 0
            background-color: #999
            position: absolute
            bottom: 0
            right: 0
            transition: all .2s
            &:hover
              background-color: #c33
            &:focus-visible
              border: 1px dotted #fff
            &::before
              content: 'delete'
          &.disabled
            opacity: 0.25
            .disable-word-btn
              &::before
                content: 'visibility_off'
              &:hover
                &::before
                  content: 'visibility'
  .add-cat-btn
    border: 0
    background-color: #666
    color: #fff
    line-height: 1
    font-size: 2rem
    padding: 1rem
    height: 4rem
    border-radius: 2rem
    &::before
      content: 'add'
.result
  position: fixed
  bottom: 0
  left: 0
  z-index: 1
  width: 100%
  padding: 0.5rem
  background-color: #fc3
  #spell
    font-size: 2rem
    background-color: #fff
    border: 0
    border-radius: 1rem
    color: #333
    padding: 1rem
    height: 4rem
    width: 100%
    display: block
</style>

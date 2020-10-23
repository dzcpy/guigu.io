<template>
  <div class="select">
    <span
      v-for="(tag, index) in tags"
      :key="tag.name"
      :style="{ backgroundColor: '#' + tag.hex }"
      class="tag"
    >
      {{ tag.name }} <span class="close" @click="clickTag(index)">✕</span>
    </span>
    <input type="text" @keyup="onInputKeyup($event.target.value, $event.key)" />
  </div>
</template>

<script>
import allColors from '../mocks/colors'
export default {
  data: () => {
    return {
      allColors,
      tags: [],
    }
  },
  methods: {
    clickTag(index) {
      this.tags.splice(index, 1)
    },
    onInputKeyup(name, key) {
      if (key === 'Enter') {
        let tagData = { name, hex: false || '#AAA' }
        this.allColors.forEach((item) => {
          console.log(item)
          if (item.name.toLowerCase() === name.toLowerCase()) {
            tagData = { ...item, name }
          }
        })
        this.tags.push(tagData)
      }
    },
  },
}
</script>

<style scoped>
.select {
  border: 1px solid #aaa;
  border-radius: 5px;
  height: 35px;
  line-height: 30px;
  width: 350px;
  text-align: left;
  vertical-align: center;
  overflow: hidden;
  cursor: text;
}
.select input {
  border: none;
  border-radius: 5px;
  line-height: 24px;
}
.select input:focus {
  outline: none;
}
.select .tag {
  display: inline-block;
  cursor: default;
  margin: 0 3px;
  border-radius: 4px;
  height: 25px;
  vertical-align: center;
  font-size: 13px;
  line-height: 24px;
  padding: 0 5px;
  user-select: none;
  color: white;
  background-color: #aaa;
}
.select .tag .close {
  cursor: pointer;
}
</style>

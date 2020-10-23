<template>
  <div class="select" @click="clickSelect">
    <span
      v-for="(tag, index) in tags"
      :key="tag.name + index"
      :style="{ backgroundColor: '#' + tag.hex }"
      class="tag"
    >
      {{ tag.name }} <span class="close" @click="clickTag(index)">✕</span>
    </span>
    <input
      ref="input"
      :style="{ marginLeft: tags.length ? '0' : '10px' }"
      type="text"
      @keyup="onInputKeyup($event.target, $event.key)"
      @blur="onInputKeyup($event.target, 'Enter')"
    />
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
    clickSelect() {
      this.$refs.input.focus()
    },
    clickTag(index) {
      this.tags.splice(index, 1)
    },
    onInputKeyup(input, key) {
      const name = input.value
      if (!name.trim()) {
        return
      }
      let tagData
      switch (key) {
        case 'Enter':
          tagData = { name, hex: false || '#AAA' }
          this.allColors.forEach((item) => {
            if (item.name.toLowerCase() === input.value.toLowerCase()) {
              tagData = { ...item, name }
            }
          })
          input.value = ''
          this.tags.push(tagData)
          break
        case 'Escape':
          input.value = ''
          break
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
  width: 370px;
  text-align: left;
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

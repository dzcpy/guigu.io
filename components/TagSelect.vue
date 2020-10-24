<template>
  <div class="tag-select">
    <div ref="select" class="select" @click="clickSelect">
      <span
        v-for="(tag, index) in selectedTags"
        :key="`${tag.name}-${index}`"
        :style="{ backgroundColor: '#' + tag.hex }"
        class="tag"
        @click.prevent.stop=""
      >
        {{ tag.name }}
        <span class="close" @click.stop.prevent="removeTag(index)">✕</span>
      </span>
      <input
        ref="input"
        type="text"
        class="input"
        @input.passive="resizeInput"
        @keyup="onInputKeyup($event.key)"
        @blur="onBlur"
      />
      <span ref="shadow" class="shadow-input"></span>
    </div>
    <transition name="fade">
      <div v-show="showDropdown" class="dropdown">
        <ul ref="ul">
          <li
            v-for="(tag, index) in tagList"
            :key="`${tag.name}-${index}`"
            :class="{ selected: tag.selected }"
            @click="selectTag(tag)"
          >
            <span :style="{ color: '#' + tag.hex }">■</span>
            {{ tag.name }}
            <span v-show="tag.selected" class="checked"></span>
          </li>
        </ul>
      </div>
    </transition>
  </div>
</template>

<script>
import allColors from '../mocks/colors'

const findName = (list, name) =>
  list.find(
    (item) => item.name.trim().toLowerCase() === name.trim().toLowerCase()
  )

export default {
  data: () => {
    return {
      allColors,
      selectedTags: [],
      tagList: [],
      showDropdown: false,
    }
  },
  methods: {
    clickSelect() {
      const element = this.$refs.input
      element.focus()
      this.openDropdown()
      if (element.value === '') {
        element.style.width = '3em'
      }
    },
    removeTag(index) {
      this.selectedTags.splice(index, 1)
    },
    onBlur() {
      this.showDropdown = false
    },
    onInputKeyup(key) {
      const name = this.$refs.input.value
      if (!name && key !== 'Backspace' && key !== 'Delete') {
        return
      }
      let tagData
      switch (key) {
        case 'Enter':
          tagData = { name, hex: false || '#AAA' }
          this.allColors.forEach((color) => {
            if (color.name.toLowerCase() === name.toLowerCase()) {
              tagData = { ...color }
            }
          })
          this.$refs.input.value = ''
          this.selectedTags.push(tagData)
          break
        case 'Escape':
          this.$refs.input.value = ''
          this.showDropdown = false
          break
        case 'Backspace':
          if (!name) {
            this.selectedTags.pop()
          }
          this.openDropdown()
          break
        default:
          this.openDropdown()
      }
    },
    openDropdown() {
      this.tagList = []
      const input = this.$refs.input.value
      this.showDropdown = true
      const selectedItems = []
      const unselectedItems = []
      if (typeof input === 'string' && input.length) {
        const tagsFound = this.allColors.filter(
          (color) =>
            color.name.toLowerCase().includes(input.trim().toLowerCase()) &&
            !findName(selectedItems, color.name.toLowerCase())
        )
        if (!tagsFound.length) {
          this.tagList.push({
            name: input,
            selected: false,
          })
        }
        tagsFound.forEach((color) => {
          this.tagList.push({
            name: color.name,
            hex: color.hex,
            selected: false,
          })
        })
        this.allColors.forEach((color) => {
          if (findName(this.selectedTags, color.name)) {
            this.tagList.push({
              name: color.name,
              hex: color.hex,
              selected: true,
            })
          }
        })
      } else {
        this.allColors.forEach((color) => {
          if (findName(this.selectedTags, color.name)) {
            selectedItems.push(color)
          } else {
            unselectedItems.push(color)
          }
        })
        this.allColors.forEach((color) =>
          this.tagList.push({
            name: color.name,
            hex: color.hex,
            selected: !!findName(selectedItems, color.name),
          })
        )
      }
    },
    selectTag(tag) {
      if (tag) {
        const nameItem = findName(this.selectedTags, tag.name)
        if (nameItem) {
          this.selectedTags.splice(this.selectedTags.indexOf(nameItem), 1)
        } else {
          this.selectedTags.push(tag)
        }
      }
      this.$refs.input.value = ''
    },
    elementIsOverflown(element) {
      return (
        element.scrollHeight > element.clientHeight ||
        element.scrollWidth > element.clientWidth
      )
    },
    resizeInput() {
      const elementInput = this.$refs.input
      const elementShadow = this.$refs.shadow
      const fontSize = parseInt(window.getComputedStyle(elementInput).fontSize)
      elementShadow.textContent = elementInput.value
      if (this.elementIsOverflown(elementInput)) {
        elementInput.style.width =
          elementInput.offsetWidth + fontSize * 3 + 'px'
      } else {
        elementInput.style.width =
          elementShadow.offsetWidth + fontSize * 3 + 'px'
      }
      this.resizeSelect()
    },
    resizeSelect(recheck = true) {
      const element = this.$refs.select
      if (this.elementIsOverflown(element)) {
        element.style.height = element.scrollHeight + 7 + 'px'
      } else if (recheck) {
        element.style.height = '35px'
        this.$nextTick(() => this.resizeSelect(false))
      }
    },
  },
  watch: {
    selectedTags() {
      this.$nextTick(() => this.resizeSelect())
    },
  },
}
</script>

<style lang="scss" scoped>
$select-width: 370px;
.tag-select {
  font-size: 12px;
}
.select {
  border: 1px solid #aaa;
  border-radius: 5px;
  height: 35px;
  padding: 0 5px 0 0;
  width: $select-width;
  text-align: left;
  overflow: hidden;
  cursor: text;
  input,
  .shadow-input {
    border: none;
    width: 3em;
    max-width: calc(#{$select-width} - 12px);
    display: inline-block;
    overflow: visible;
    padding: 0 5px;
    margin: 5px 0 0 5px;
    border-radius: 5px;
    line-height: 23px;
  }
  input:focus {
    outline: none;
  }
  .shadow-input {
    width: auto !important;
    opacity: 0 !important;
    position: absolute !important;
    left: -99999em !important;
    top: -99999em !important;
  }
  .tag {
    display: inline-block;
    cursor: default;
    margin: 5px 0 0 5px;
    border-radius: 4px;
    height: 23px;
    line-height: 23px;
    padding: 0 5px;
    user-select: none;
    color: #fff;
    background-color: #aaa;
    &:last-of-type {
      margin-right: -8px;
    }
    .close {
      cursor: pointer;
    }
  }
}
.dropdown {
  width: $select-width;
  max-height: 250px;
  z-index: 9999;
  position: absolute;
  overflow: auto;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 3px;
  padding: 0;
  margin: 0;
  margin-top: 3px;
  box-shadow: 1px 1px 5px 2px rgba(0, 0, 0, 0.1);
  ul {
    text-align: left;
    list-style: none;
    margin: 0;
    padding: 0;
    outline: none;
    li {
      margin: 0;
      list-style: none;
      padding: 0 5px;
      cursor: pointer;
      overflow: hidden;
      &:hover {
        background-color: #e6f7ff !important;
      }
      span.checked::after {
        content: '✓';
        float: right;
        margin-right: 5px;
      }
      &:hover span.checked::after {
        color: #1890ff;
        content: '🞩';
      }
      .selected {
        background-color: #eee;
      }
    }
  }
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>

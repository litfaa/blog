<template>
  <div class="err-content">
    <div class="center">
      <h1>{{ code }}</h1>
      <p>{{ msg[msgIndex] }}</p>
      <a href="/">返回首页</a>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    code: {
      type: [String, Number],
      default: '404'
    }
  },
  computed: {
    msgIndex() {
      return Math.floor(Math.random() * this.msg.length)
      // return Math.random()
    }
  },
  data() {
    return {
      // status: 404,
      msg: ['How did we get here?', "Looks like we've got some broken links.", "That's a Four-Oh-Four.", "Looks like we've got some broken links.", "There's nothing here."]
    }
  }
}
</script>

<style lang="less" scoped>
.err-content {
  width: 100%;
  height: 80vh;
  position: relative;
  .center {
    width: 375px;
    height: 100px;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    margin: auto;
  }
}
h1 {
  color: #2c3e50;
}
p {
  font-size: 1rem;
  color: #999;
  border-left: 0.2rem solid #dfe2e5;
  margin: 1rem 0;
  padding: 0.25rem 0 0.25rem 1rem;
}
a {
  color: #3eaf7c;
  font-weight: 500;
  text-decoration: none;
}
</style>

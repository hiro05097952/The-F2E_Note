<template>
  <router-view/>
</template>

<script>
export default {
  name: 'App',
};
</script>

<style lang="scss">
@import './sass/_reset.scss';
@import 'quill/dist/quill.core.css';
@import 'quill/dist/quill.snow.css';
@import 'quill/dist/quill.bubble.css';
.quill-editor{
  height: 90%;
  position: relative;
  .ql-toolbar{
    border: none;
    border-bottom: 1px solid #ccc;
    span.ql-formats{
      margin-top: 10px;
    }
  }
  .ql-container{
    border: none;
    padding-top: 30px;
    overflow: auto;
    height: 400px;
  }
}
.content.moon{
  .quill-editor{
    .ql-toolbar{
      span.ql-formats{
        filter: invert(100%);
        .ql-picker-options{
          background-color: rgb(205, 205, 206);
          box-shadow: none;
          border: 1px solid #A3A3A3;
        }
      }
    }
    .ql-container{
      color: #F2F2F2;
    }
  }
}
</style>

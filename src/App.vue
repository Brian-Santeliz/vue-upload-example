<template>
  <div>
    <h1>Agrega imagenes</h1>
    <input type="file" name="" id="" @change="addChange" />
    <button type="submit" @click="subir" :disabled="disabled">Subit</button>
  </div>
</template>

<script>
import uploadImage from "../graphql/uploadImage.graphql";
export default {
  name: "App",
  data() {
    return {
      file: null,
      disabled: false,
    };
  },
  methods: {
    addChange({ target }) {
      this.file = target.files[0];
    },
    subir() {
      if (!this.file) return;
      this.disabled = true;
      this.$apollo
        .mutate({
          mutation: uploadImage,
          variables: {
            file: this.file,
          },
        })
        .then(
          ({
            data: {
              singleUpload: { filename },
            },
          }) => {
            console.log(filename);
          }
        );
      this.disabled = false;
    },
  },
};
</script>

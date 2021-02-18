<template>
  <div>
    <h1>Agrega imagenes</h1>
    <input type="file" name="" accept="image/*" id="" @change="addChange" />
    <input
      type="file"
      name=""
      id=""
      ref="inputFiles"
      @change="addMultiple"
      accept="image/*"
      multiple
    />
    <button type="submit" @click="subir" :disabled="disabled">
      Submit Single
    </button>
    <button type="submit" @click="subirArray" :disabled="disabled">
      Submit ARRAY
    </button>
    <div v-for="(upload, index) in allUpload" :key="index">
      <img :src="upload.path" alt="" />
      <p>{{ upload.filename }}</p>
    </div>
    <h1>MUTACION DE EJEMPLO CON TRACKING UQER</h1>
    <input type="file" name="" id="" @change="handleChangeTracking" />
    <input type="file" name="" id="" multiple @change="handleChangeTracking" />
  </div>
</template>

<script>
import uploadImage from "../graphql/uploadImage.graphql";
import subidaFoto from "../graphql/subidaFoto.gql";
import arrayImagenCustom from "../graphql/arrayImagenCustom.gql";
import fotoEntranda from "../graphql/fotoEntranda.graphql";
// import uploadArrayImages from "../graphql/uploadArrayImages.graphql";
import getImages from "../graphql/getArrayImage.graphql";

export default {
  name: "App",
  data() {
    return {
      file: null,
      disabled: false,
      files: [],
      fileTracking: null,
      filesTrackingArray: [],
    };
  },
  apollo: {
    allUpload: getImages,
  },
  methods: {
    addChange({ target }) {
      this.file = target.files[0];
    },
    addMultiple({ target }) {
      // this.files = this.$refs.inputFiles.files;
      this.files = target.files;
    },
    subir() {
      if (!this.file) {
        return;
      }
      this.disabled = true;
      this.$apollo
        .mutate({
          mutation: uploadImage,
          variables: {
            file: this.file,
          },
          update: (store, { data: { singleUpload } }) => {
            const { allUpload } = store.readQuery({
              query: getImages,
            });
            store.writeQuery({
              query: getImages,
              data: {
                allUpload: [...allUpload, singleUpload],
              },
            });
          },
        })
        .then(({ data: { singleUpload } }) => {
          this.disabled = false;
          console.log(singleUpload);
        })
        .catch((e) => console.log(e));
    },
    handleChangeArrayTrackin({ target }) {
      this.filesTrackingArray = target.files;
      this.$apollo
        .mutate({
          mutation: fotoEntranda,
          variables: {
            fotos: this.filesTrackingArray,
          },
        })
        .then((res) => console.log(res));
    },
    handleChangeTracking({ target }) {
      this.fileTracking = target.files[0];
      this.$apollo
        .mutate({
          mutation: subidaFoto,
          variables: {
            foto: this.fileTracking,
          },
        })
        .then((r) => console.log(r));
    },
    subirArray() {
      this.disabled = true;
      this.$apollo
        .mutate({
          mutation: arrayImagenCustom,
          variables: {
            files: { fotos: this.files },
          },
          update: (store, { data: { arrayUpload } }) => {
            const data = store.readQuery({
              query: getImages,
            });
            console.log("resultacdo de la query", data);
            store.writeQuery({
              query: getImages,
              data: { allUpload: arrayUpload },
            });
          },
        })
        .then(() => {
          this.disabled = false;
        });
    },
  },
};
</script>

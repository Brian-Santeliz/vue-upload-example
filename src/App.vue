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
    <input
      type="file"
      name=""
      id=""
      multiple
      @change="handleChangeArrayTrackin"
    />
    <p>INPUT DEFINITIVO</p>
    <input type="file" name="" id="" multiple @change="hanldeDefinitivo" />
    <input type="file" name="" id="" multiple @change="ManejadorCambios" />
    <br /><br /><br /><br />
    <input type="file" name="" id="" multiple @change="handlePaqueteCrear" />
  </div>
</template>

<script>
import uploadImage from "../graphql/uploadImage.graphql";
// import arrefloFotosTracking from "../graphql/arrefloFotosTracking.graphql";
import subidaFoto from "../graphql/subidaFoto.gql";
import fotoEntradaConUploadArray from "../graphql/fotoEntradaConUploadArray.gql";
import fotoEntradaArrayOriginal from "../graphql/fotoEntradaArrayOriginal.gql";
import arrayImagenCustom from "../graphql/arrayImagenCustom.gql";
import paqueteCrear from "../graphql/paqueteCrear.gql";
import fotoDefinitiva from "../graphql/fotoDefinitiva.gql";
// import fotoEntranda from "../graphql/fotoEntranda.graphql";
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
    ManejadorCambios({ target }) {
      this.filesTrackingArray = target.files;
      const foto = {
        foto: this.filesTrackingArray,
      };
      this.$apollo
        .mutate({
          mutation: fotoEntradaArrayOriginal,
          variables: {
            fotos: [foto],
          },
        })
        .then((res) => console.log(res));
    },
    handleChangeArrayTrackin({ target }) {
      this.filesTrackingArray = target.files;
      const foto = {
        foto: this.filesTrackingArray,
      };
      this.$apollo
        .mutate({
          mutation: fotoEntradaConUploadArray,
          variables: {
            FotoEntrada: {
              fotos: [foto],
            },
          },
        })
        .then((res) => console.log(res));
    },
    hanldeDefinitivo({ target }) {
      this.filesTrackingArray = target.files;
      const foto = {
        foto: this.filesTrackingArray,
      };
      console.log(foto);
      this.$apollo
        .mutate({
          mutation: fotoDefinitiva,
          variables: {
            fotos: {
              foto: [foto],
              id: "2121",
              url: "23232",
            },
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
            FotoEntrada: {
              foto: this.fileTracking,
            },
          },
        })
        .then((r) => console.log(r));
    },
    handlePaqueteCrear({ target }) {
      //       if (o.files.length == 0 || !(/\.(jpg|png)$/i).test(foto.name)) {
      //   alert('Ingrese una imagen con alguno de los siguientes formatos: .jpeg/.jpg/.png.');
      //   return false;
      // }
      //              if (!this.filesTrackingArray.length && !(this.filesTrackingArray.type == 'image/png' || this.filesTrackingArray.type == 'image/jpg' ){
      //         return;
      // },
      this.filesTrackingArray = target.files;
      let extension = false;
      let peso = false;
      this.filesTrackingArray.forEach((file) => {
        //Valida el formato de la imagen
        const result = /\.(jpg|jpeg|png)$/i.test(file.name);
        return result
          ? (extension = true)
          : ((extension = false), alert("El archivo no es una imagen"));
      });
      this.filesTrackingArray.forEach((file) => {
        //Validar lo que es el peso de la imagen
        return file.size > 50000
          ? ((peso = false), alert("La imagen debe pesar menos de 500kb"))
          : (peso = true);
      });
      if (extension && peso) {
        const foto = {
          foto: this.filesTrackingArray,
        };
        this.$apollo
          .mutate({
            mutation: paqueteCrear,
            variables: {
              paquete: {},
              fotos: {
                foto: [foto],
              },
              personaId: "2232",
            },
          })
          .then((res) => console.log(res))
          .catch((e) => console.log(e));
      }
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

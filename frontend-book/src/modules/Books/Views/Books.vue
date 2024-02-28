<template>
  <div>
    <div>
      <b-carousel
        style="text-shadow: 0px 0px 2px #000"
        indicators
        img-width="500"
        img-height="500"
      >
        <b-carousel-slide
          v-for="(book, index) in data"
          :key="index"
          :caption="book.name"
          :img-src="base64ToImage(book.cover)"
          class="carrusel"
        ></b-carousel-slide>
      </b-carousel>
    </div>

    <div>
      <b-row class="p-3">
      <b-col cols="5">
        <b-datepicker v-model="fechaInicio"></b-datepicker>
      </b-col>
      <b-col cols="5">
        <b-datepicker v-model="fechaFin"></b-datepicker>
      </b-col>
      <b-col cols="2">
        <b-button variant="primary" @click="getDate">Buscar libros</b-button>
      </b-col>
    </b-row>
    <b-row class="p-3" align-v="end">
      <b-col cols="12">
        <b-input-group label="Fecha de inicio">
          <template>
            <b-dropdown v-model="tipoBusqueda" text="categorys" variant="primary">
              <b-dropdown-item @click="tipoBusqueda = 'autor'">Autor</b-dropdown-item>
              <b-dropdown-item @click="tipoBusqueda = 'genero'">Género</b-dropdown-item>
              <b-dropdown-item @click="tipoBusqueda = 'nombre'">Nombre</b-dropdown-item>
            </b-dropdown>
          </template>
          <b-form-input v-model="consulta" placeholder="Ingrese su consulta..."/>
          <b-input-group-append>
            <b-button variant="primary" @click="getCategory">Buscar</b-button>
          </b-input-group-append>
        </b-input-group>
      </b-col>
      <b-col>
      </b-col>
    </b-row>
      <div class="bodybuttons">
        <b-button v-b-modal.modal-save> Registrar libro </b-button>
      </div>
    </div>

    <div class="d-flex flex-wrap justify-content-around">
      <b-col v-for="(book, index) in data" :key="index" class="d-flex d-fixed">
        <b-card style="height: 100%; width: auto">
          <b-card-img :src="base64ToImage(book.cover)"></b-card-img>
          <b-card-title>{{ book.name }}</b-card-title>
          <b-card-sub-title>{{ book.autor }}</b-card-sub-title>
          <b-card-text>{{ book.publishDate }}</b-card-text>
          <template #footer>
            <div>
              <b-button variant="faded" @click="OpenEditModal(book)"
                >Editar</b-button
              >
              <b-button
                variant="faded"
                style="color: red"
                @click="deleteBook(book.id)"
                >Eliminar</b-button
              >
            </div>
          </template>
        </b-card>
      </b-col>
      <b-col class="iconos">
        <br />
        <div
          class="drop-zone"
          @drop="OpenEditModal($event)"
          @dragover.prevent
          @dragenter.prevent
        >
          ARRASTRA PARA EDITAR
        </div>
        <br />
        <div
          class="drop-zone"
          @drop="deleteBook($event)"
          @dragover.prevent
          @dragenter.prevent
        >
          ARRASTRA PARA ELIMINAR
        </div>
      </b-col>
    </div>

    <ModalSave @book-updated="fetchData" />
    <ModalUpdate
      ref="ModalUpdate"
      :book="selectedBook"
      @book-updated="fetchData"
    />
  </div>
</template>

<script>
import ModalSave from "./Modal-save.vue";
import ModalUpdate from "./Modal-update.vue";
import axios from "axios";
import Swal from "sweetalert2";

export default {
  components: { ModalSave, ModalUpdate },
  name: "Books",
  data() {
    return {
      data: null,
      selectedBook: null,
      book: {
        id: "",
        name: "",
        autor: "",
        publishDate: null,
      },
    };
  },
  computed: {},
  methods: {
    showModal() {
      this.show = true;
    },
    base64ToImage(base64String) {
      console.log("Valor de base64String:", base64String);
      const type = base64String.substring(
        "data:image/".length,
        base64String.indexOf(";base64")
      );

      const byteCharacters = atob(base64String.split(",")[1]);
      const byteArrays = [];
      for (let offset = 0; offset < byteCharacters.length; offset += 512) {
        const slice = byteCharacters.slice(offset, offset + 512);
        const byteNumbers = new Array(slice.length);
        for (let i = 0; i < slice.length; i++) {
          byteNumbers[i] = slice.charCodeAt(i);
        }
        const byteArray = new Uint8Array(byteNumbers);
        byteArrays.push(byteArray);
      }
      const blob = new Blob(byteArrays, { type: type });
      const url = URL.createObjectURL(blob);
      return url;
    },
    fetchData() {
      axios
        .get("http://localhost:8080/api-book/")
        .then((response) => {
          this.data = response.data.data;
        })
        .catch((error) => {
          console.error("Error al obtener datos de la API", error);
        });
    },
    filterMovies() {
      this.currentPage = 1;
      this.fetchData();
    },
    OpenEditModal(book) {
      this.selectedBook = book;
      this.$bvModal.show("modal-update");
    },
    async deleteBook(id) {
      const confirmed = await Swal.fire({
        title: "¿Estás seguro de eliminar el libro?",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#008c6f",
        cancelButtonColor: "#e11c24",
        confirmButtonText: "Sí, eliminar",
        cancelButtonText: "Cancelar",
      });

      if (confirmed.isConfirmed) {
        try {
          const response = await axios.delete(
            `http://localhost:8080/api-book/libro/${id}`
          );
          if (response.data.error) {
            console.error(response.data.message);
          } else {
            Swal.fire({
              title: "Eliminado",
              text: "El libro se elimino correctamente",
              icon: "success",
              timer: 3000,
            });
            this.fetchData();
          }
        } catch (error) {
          const { data } = error;
          this.$swal.fire({
            icon: "error",
            text: data?.text ? data.text : "Error interno",
            timer: 3000,
          });
        }
      }
    },

    startDrag(evt, item) {
      evt.dataTransfer.dropEffect = "move";
      evt.dataTransfer.effectAllowed = "move";
      evt.dataTransfer.setData("itemID", item.id);
    },
  },
  mounted() {
    this.fetchData();
  },
};
</script>

<style>
.bodybuttons {
  text-align: left;
  padding-left: 20px;
  padding-top: 20px;
  padding-bottom: 20px;
}

.iconos {
  text-align: right;
  padding-right: 20px;
  padding-top: 20px;
  padding-bottom: 20px;
}

.carrusel {
  max-height: 400px;
}
</style>

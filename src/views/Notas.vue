<template>
  <div class="container">
    <h1>Notas</h1>

    <b-alert
      :show="dismissCountDown"
      dismissible
      :variant="mensaje.color"
      @dismissed="dismissCountDown=0"
      @dismiss-count-down="countDownChanged"
    >{{this.mensaje.texto}}</b-alert>

    <form @submit.prevent="agregarNota()">
      <input type="text" placeholder="Nombre" class="form-control my-2" v-model="nota.nombre" />
      <input
        type="text"
        placeholder="Descripcion"
        class="form-control my-2"
        v-model="nota.descripcion"
      />
      <b-button class="btn-success my-2 btn-block" type="submit">Agregar Nota</b-button>
    </form>

    <table class="table">
      <thead>
        <tr>
          <th scope="col">#</th>
          <th scope="col">Nombre</th>
          <th scope="col">Descripción</th>
          <th scope="col">Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item,index) in notas" :key="index">
          <th scope="row">{{item._id}}</th>
          <td>{{item.nombre}}</td>
          <td>{{item.descripcion}}</td>
          <td>
            <b-button @click="eliminarNota(item._id)" class="btn-danger">Acción</b-button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      notas: [],
      dismissSecs: 5,
      dismissCountDown: 0,
      mensaje: {
        color: "",
        texto: ""
      },
      nota: {
        nombre: "",
        descripcion: ""
      }
    };
  },
  created() {
    this.listarNotas();
  },
  methods: {
    listarNotas() {
      this.axios
        .get("/nota")
        .then(res => {
          console.log(res.data);
          this.notas = res.data;
        })
        .catch(e => {
          console.log(e.response);
        });
    },
    alerta() {
      this.mensaje.color = "danger";
      this.mensaje.texto = "Probanado Alerta";
      this.showAlert();
    },
    agregarNota() {
      console.log(this.nota);
      this.axios
        .post("/nueva-nota", this.nota)
        .then(res => {
          this.notas.push(res.data);
          this.nota.nombre = "";
          this.nota.descripcion = "";
          this.mensaje.color = "success";
          this.mensaje.texto = "Nota Agregada!";
          this.showAlert();
        })
        .catch(e => {
          console.log(e.response);
          console.log(e.response.data.error.errors.nombre.message);
          if (e.response.data.error.errors.nombre.message) {
            this.mensaje.texto = e.response.data.error.errors.nombre.message;
          } else {
            this.mensaje.texto = "Error de sistema";
          }

          this.mensaje.color = "danger";
          this.showAlert();
        });
    },
    eliminarNota(id) {
      console.log(id);
      this.axios
        .delete(`/nota/${id}`)
        .then(res => {
          const index = this.notas.findIndex(item => item._id === res.data._id);
          this.notas.splice(index, 1);
          this.mensaje.color = "success";
          this.mensaje.texto = "Nota Eliminada!";
          this.showAlert();
        })
        .catch(e => {
          console.log(e.response);
          this.mensaje.color = "danger";
          this.mensaje.texto = "Error de sistema";
          this.showAlert();
        });
    },
    countDownChanged(dismissCountDown) {
      this.dismissCountDown = dismissCountDown;
    },
    showAlert() {
      this.dismissCountDown = this.dismissSecs;
    }
  }
};
</script>
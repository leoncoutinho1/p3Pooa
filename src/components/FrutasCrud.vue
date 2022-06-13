<template>
<div>
    <v-data-table
        :headers="headers"
        :items="frutas"
        sort-by="calories"
        class="elevation-1"
    >   
        <template v-slot:[`item.actions`]="{ item }">
            <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
            <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>
        <template v-slot:no-data>
            <v-btn color="primary" @click="initialize"> Reset </v-btn>
        </template>
    </v-data-table>
    <v-btn color="primary" elevation="2" outlined plain @click="this.showForm">Novo registro</v-btn>
    <v-form
        ref="form"
        v-show="this.dialog">
            <v-container>
                <v-row>
                    <v-col cols="12" sm="4">
                        <v-text-field
                        v-model="editedItem.nome"
                        :rules="rulesNome"
                        label="Nome"
                        required></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="4">
                        <v-text-field
                        v-model="editedItem.calorias"
                        label="Quantidade de calorias"
                        :rules="rulesCalorias"
                        required></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="4">
                        <v-btn color="primary" elevation="2" outlined plain @click="this.save">Adicionar</v-btn>
                    </v-col>
                </v-row>
            </v-container>
    </v-form>
    </div>
</template>

<script>
import axios from "axios";
export default {
  name: "FrutasCrud",
  data: () => ({
    dialog: false,
    headers: [
      { text: "Id", value: "id" },
      { text: "Nome", value: "nome" },
      { text: "Calorias", value: "calorias" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    frutas: [],
    editedIndex: -1,
    editedItem: {
      id: 0,
      nome: "",
      calorias: 0,
    },
    defaultItem: {
      id: 0,
      nome: "",
      calorias: 100,
    },
    rulesCalorias: [
        v => (v && v > 0) || "Deve ser um valor numérico"
    ],
    rulesNome: [
        v => (v.length > 2 && typeof(v) === 'string') || "O nome deve ter ao menos três caracteres"
    ]
  }),
  methods: {
    initialize() {
      axios("http://localhost:3000/frutas")
        .then((response) => {
          this.frutas = response.data;
        })
        .catch((error) => console.log(error));
    },
    close() {
      this.dialog = false;
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      }, 300);
    },
    save() {
        if (this.$refs.form.validate()) {
            if (this.editedIndex > -1) {
                //alteracao
                axios
                .put(
                    "http://localhost:3000/frutas/" + this.editedItem.id,
                    this.editedItem
                )
                .then((response) => {
                    console.log(response);
                    Object.assign(this.frutas[this.editedIndex], this.editedItem);
                    this.close();
                })
                .catch((error) => console.log(error));
            } else {
                //Inclusao
                axios
                .post("http://localhost:3000/frutas", this.editedItem)
                .then((response) => {
                    this.frutas.push(response.data);
                    this.close();
                })
                .catch((error) => console.log(error));
            }
        } else {
            console.log("Preencha corretamente os campos")
        }
    },
    editItem(item) {
      this.editedIndex = this.frutas.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    deleteItem(item) {
        if (confirm(`Deseja realmente remover o item ${item.nome}?`)) {
            axios
                .delete("http://localhost:3000/frutas/"+item.id)
                .then((response) => {
                    if (response.status === 200) {
                        this.frutas = this.frutas.filter(i => i.id != item.id)
                    }
                })
                .catch((error) => console.log(error));
        }
    },
    showForm() {
        this.dialog = !this.dialog
    }
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Novo Item" : "Editar Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.initialize();
  },
};
</script>

<style scoped>
</style>
<template>
  <q-page padding>
    <q-layout>
      <div class="row reverse">
        <q-btn icon="add" @click.native="toggleModal('novaCrianca')">&nbsp;</q-btn>
      </div>
      <div class="row my-1">
        <div class="col">
          <q-field>
            <q-input v-model="searchWords" placeholder="Digite aqui pra pesquisar..."/>
          </q-field>
        </div>
      </div>
      <q-table
        title="Crianças"
        :data="lista"
        :columns="headers"
        :filter="searchWords"
        row-key="uid"
      />
    </q-layout>
    <!-- Adicionar Criança -->
    <q-modal v-model="modals.novaCrianca" content-css="padding: 40px; min-width: 32rem">
      <h4 style="margin-top: 0; margin-bottom: 10px; font-size: 2rem">Registrar criança</h4>
      <q-field label="Nome">
        <q-input v-model="form.nome"/>
      </q-field>
      <q-field label="Nascimento">
        <q-input v-model="form.nascimento" v-mask="'##/##/####'" :error="form.invaliddate"/>
      </q-field>
      <q-field label="Responsável">
        <q-input v-model="form.responsavel"/>
      </q-field>
      <q-field label="Telefone">
        <q-input v-model="form.telefone" v-mask="'(##) #####-####'"/>
      </q-field>
      <div class="row reverse" style="padding-top: 20px">
        <q-btn flat @click.native="resetForm()">Redefinir</q-btn>
        <q-btn flat @click.native="addCrianca(form)" :disable="!formvalid">Salvar</q-btn>
      </div>
    </q-modal>
    <!-- Fim Adicionar Criança -->
  </q-page>
</template>

<script>
import { date } from 'quasar'
import { mask } from 'vue-the-mask'

const headers = [
  { align: 'left', sortable: true, name: 'nome', field: 'nome', label: 'Nome' },
  { align: 'left', sortable: true, name: 'nascimento', field: 'nascimento', label: 'Nascimento' },
  { align: 'left', sortable: true, name: 'responsavel', field: 'responsavel', label: 'Responsavel' },
  { align: 'left', sortable: true, name: 'telefone', field: 'telefone', label: 'Telefone' }
]

export default {
  name: 'Lista',
  directives: { mask },
  beforeMount: function () {
    const saved = localStorage.getItem('lista')
    if (saved) {
      this.lista = JSON.parse(saved) || []
    }
  },
  computed: {
    formvalid: function () {
      const nome = this.form.nome && this.form.nome.length > 5
      const date = !this.form.invaliddate
      const resp = this.form.responsavel && this.form.responsavel.length > 5
      const tel = this.form.telefone && this.form.telefone.length >= 14
      return nome && date && resp && tel
    }
  },
  watch: {
    'form.nascimento': function (newVal, oldVal) {
      if (newVal && oldVal && newVal !== oldVal && newVal.length >= 10) {
        const pieces = newVal.split('/')
        const valid = date.isValid(`${pieces[2]}-${pieces[1]}-${pieces[0]}`)
        if (!valid) {
          this.form.invaliddate = true
        }
      } else if (newVal && oldVal && newVal !== oldVal && newVal.length < 10) {
        this.form.invaliddate = false
      }
    }
  },
  methods: {
    toggleModal (modalName) {
      // Abrir se estiver fechado, e fechar se estiver aberto.
      this.modals[modalName] = !this.modals[modalName]
    },
    addCrianca: function (form) {
      const uid = Math.random()
      const { nome, nascimento, responsavel, telefone } = form
      this.lista.push({ uid, nome, nascimento, responsavel, telefone })
      this.form.nome = null
      this.form.nascimento = null
      this.form.invaliddate = false
      this.form.responsavel = null
      this.form.telefone = null
      localStorage.setItem('lista', JSON.stringify(this.lista))
      this.toggleModal()
    }
  },
  data () {
    return {
      searchWords: '',
      modals: {
        novaCrianca: false
      },
      lista: [],
      form: {
        nome: null,
        nascimento: null,
        invaliddate: false,
        responsavel: null,
        telefone: null
      },
      headers
    }
  }
}
</script>

<style>
</style>

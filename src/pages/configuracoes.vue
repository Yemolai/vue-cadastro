<template>
  <q-page padding>
    <q-layout>
      <div>
        <!-- Icon & Label -->
        <q-btn color="red" icon="delete" @click.native="toggleModal('confirmaExcluir')" label="Apagar os dados" />
      </div>
      <div class="q-mt-md">
        <p>Relatar bugs, código fonte, ajuda, dicas no repositório <a href="https://github.com/Yemolai/vue-cadastro" target="_blank">github.com/Yemolai/vue-cadastro</a></p>
      </div>
    </q-layout>

    <!-- MODALS -->
    <!-- Confirmação Apagar -->
    <q-modal v-model="modals.confirmaExcluir"  content-css="padding: 40px;">
      <h4 style="margin-top: 0; margin-bottom: 10px; font-size: 2rem">Tem certeza?</h4>
      <p>Você deseja realmente excluir a lista?</p>
      <p>Você não pode recuperá-la, a menos que tenha uma exportação salva em seu dispositivo.</p>
      <div class="row reverse" style="padding-top: 20px">
        <q-btn color="primary" @click="toggleModal('confirmaExcluir')">Cancelar</q-btn>
        <q-btn color="red" flat @click="excluirDados()">Apagar</q-btn>
      </div>
    </q-modal>
    <!-- Fim Confirmação Apagar -->
  </q-page>
</template>

<script>
export default {
  name: 'Configuracoes',
  beforeMount () {
    const saved = localStorage.getItem('lista')
    if (saved) {
      this.lista = JSON.parse(saved) || []
    }
  },
  methods: {
    //
    // FUNÇÕES
    // Abrir (ou fechar) um determinado modal
    toggleModal (modalName) {
      // Abrir se estiver fechado, e fechar se estiver aberto.
      this.modals[modalName] = !this.modals[modalName]
    },
    excluirDados () {
      localStorage.removeItem('lista')
      this.toggleModal('confirmaExcluir')
    }
  },
  data () {
    return {
      modals: {
        confirmaExcluir: false
      }
    }
  }
}
</script>

<style>
</style>

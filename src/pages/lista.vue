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

      <!-- Importar e Exportar -->
      <q-fab color="primary" icon="swap_vert" direction="up" class="fixed" style="right: 18px; bottom: 18px">
        <q-fab-action color="primary" @click="importClick" icon="open_in_browser"/>
        <q-fab-action color="primary" @click="exportClick" icon="save"/>
      </q-fab>
      <!-- Fim Importar e Exportar -->

      <!-- Necessário para subir o arquivo -->
      <input id="pegarArquivo" @change="changeImport(this)" ref="pegarArquivo" type="file" hidden>
    </q-layout>

    <!-- MODALS -->
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
        <q-btn flat @click="addCrianca(form)" :disable="!formvalid">Salvar</q-btn>
        <q-btn flat @click="resetForm()">Redefinir</q-btn>
      </div>
    </q-modal>
    <!-- Fim Adicionar Criança -->

    <!-- Opção Lista Importada -->
    <q-modal v-model="modals.oQueFazerLista" content-css="padding: 40px; min-width: 32rem">
      <h4 style="margin-top: 0; margin-bottom: 10px; font-size: 2rem">Quase lá...</h4>
      <p>Pelo visto, sua lista contém {{ lista.length }} {{ (lista.length > 1) ? "itens" : "item" }}, e você deseja importar outra.</p>
      <p>O que faremos?</p>
      <div class="row reverse" style="padding-top: 20px">
        <q-btn flat @click="respostaModalLista('mesclar')">Mescle</q-btn>
        <q-btn flat @click="respostaModalLista('substituir')">Substitua</q-btn>
      </div>
    </q-modal>
    <!-- Fim Opção Lista Importada -->

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
      const nome = this.form.nome && this.form.nome.length > 2
      const date = !this.form.invaliddate
      const resp = this.form.responsavel && this.form.responsavel.length > 2
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
    //
    // FUNÇÕES
    // Abrir (ou fechar) um determinado modal
    toggleModal (modalName) {
      // Abrir se estiver fechado, e fechar se estiver aberto.
      this.modals[modalName] = !this.modals[modalName]
    },
    // Função para salvar a lista
    setLista (lista) {
      localStorage.setItem('lista', JSON.stringify(lista))
      this.lista = lista
      return true
    },
    // Função para pegar (retornar) a lista
    getLista () {
      const saved = localStorage.getItem('lista')
      if (saved) {
        return JSON.parse(saved) || []
      }
      return []
    },
    //
    // FUNÇÕES DO MODAL DE ADICIONAR CRIANÇA
    // Função que adiciona criança
    addCrianca: function (form) {
      const uid = Math.random()
      const { nome, nascimento, responsavel, telefone } = form
      const tempLista = this.lista.concat({ uid, nome, nascimento, responsavel, telefone })
      this.setLista(tempLista)
      this.toggleModal('novaCrianca')
      this.resetForm()
    },
    // Função que limpa o formulário do modal
    resetForm () {
      this.form = {
        nome: null,
        nascimento: null,
        invaliddate: false,
        responsavel: null,
        telefone: null
      }
    },
    //
    // FUNÇÕES PARA IMPORTAR E EXPORTAR AS CRIANÇAS (não! não é um tráfico ilegal de pessoas)
    // Função de intermédio entre o clique e a importação
    importClick () {
      const pegarArquivo = document.getElementById('pegarArquivo')
      // abrir o input file
      pegarArquivo.click()
    },
    // Função de intermedio entre o clique e o download
    exportClick () {
      const lista = this.lista
      downloadJSON(lista, 'criancas')
    },
    // Houve uma seleção no input da lista
    changeImport () {
      const file = this.$refs.pegarArquivo
      const reader = readText(file)
      const _this = this
      reader.onload = function (e) {
        const arquivoLista = e.target.result
        const listaLoad = JSON.parse(arquivoLista) || []

        if (listaLoad.length > 0 && _this.lista.length > 0) {
          _this.tempLista = listaLoad
          _this.toggleModal('oQueFazerLista')
        } else if (listaLoad.length > 0 && _this.lista.length === 0) {
          _this.setLista(listaLoad)
        }
      }
    },
    // Quando clicar em qualquer opção do modal
    respostaModalLista (resposta) {
      if (resposta === 'substituir') {
        this.lista = [].concat(this.tempLista)
      } else if (resposta === 'mesclar') {
        this.lista = onlyUniqueId(this.tempLista.concat(this.lista))
      }
      this.tempLista = []
      this.toggleModal('oQueFazerLista')
      return resposta
    }
  },
  data () {
    return {
      searchWords: '',
      modals: {
        novaCrianca: false,
        oQueFazerLista: false
      },
      lista: [],
      tempLista: [],
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

// FUNÇÕES NECESSÁRIAS PARA O FUNCIONAMENTO

// Casso possua dois itens da lista com o mesmo identificador, pegar somente um... ou seja, excluir os id's repetidos.
function onlyUniqueId (obj) {
  var a = obj.concat()

  for (var i = 0; i < a.length; ++i) {
    for (var j = i + 1; j < a.length; ++j) {
      if (a[i].uid === a[j].uid) {
        a.splice(j--, 1)
      }
    }
  }
  return a
}

// Exportaão: Converter um objeto/lista em texto, e depois baixá-lo
function downloadJSON (lista, filename) {
  if (lista.length) {
    // Convertemos a lista em string e depois um encode para suporte
    const listaStr = encodeURIComponent(JSON.stringify(lista))
    const dataStr = 'data:text/json;charset=utf-8,' + listaStr
    const anchor = document.createElement('a')
    anchor.setAttribute('href', dataStr)
    anchor.setAttribute('download', filename + '.json')
    anchor.click()
    anchor.remove()
  }
}

// Importação: checar se a plataforma do usuário suporta a api para leitura de arquivos. Se possuir, atribuir essa api a variável reader
let reader = false
function checkFileAPI () {
  if (window.File && window.FileReader && window.FileList && window.Blob) {
    reader = new FileReader()
    return true
  } else {
    console.error('A API de arquivo não é totalmente suportada em seu navegador...')
  }
}

// Verificar se possui suporte
checkFileAPI()

// Importação: função que lê o texto do input
function readText (filePath) {
  if (reader !== false) {
    if (filePath.files && filePath.files[0]) {
      reader.readAsText(filePath.files[0])
      return reader
    }
  }
}

</script>

<style>
</style>

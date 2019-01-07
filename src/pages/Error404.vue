<template>
  <div class="fixed-center text-center">
    <p>
      <img
        src="~assets/sad.svg"
        style="width:30vw;max-width:150px;"
      >
    </p>
    <h5 class="text-faded">
      <strong>Desculpe, essa página não foi encontrada</strong>
    </h5>
    <p class="text-faded">
      Código do Erro: 404
    </p>
    <q-btn
      color="secondary"
      style="width:200px;"
      @click="$router.push('/')"
    >Voltar</q-btn>
  </div>
</template>

<script>
export default {
  name: 'Error404'
}
</script>

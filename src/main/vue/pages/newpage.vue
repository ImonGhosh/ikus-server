<template>
  <MainContainer title="Page in progress" icon="mdi-timer-sand">

    <template v-slot:intro>
      Eine neue Seite erscheint bald - bleibt dran!
    </template>

  </MainContainer>
</template>

<script>
import MainContainer from "@/components/layout/MainContainer";

export default {
  name: 'NewPageView',
  components: {MainContainer},
  mounted() {
    console.log("NewPageView mounted.");
  }
}
</script>
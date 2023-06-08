<template>
  <div>
    <h1>Análise de Frete</h1>
    <div>
      <label for="weight">Peso do Frete:</label>
      <input type="number" id="weight" v-model="weight" />
    </div>
    <div>
      <label for="destination">Cidade de Destino:</label>
      <input type="text" id="destination" v-model="destination" />
    </div>
    <button @click="analyzeFreight">Analisar</button>
    <div v-if="cheapestFreight">
      <h2>Frete Mais Econômico:</h2>
      <p>Transportadora: {{ cheapestFreight.name }}</p>
      <p>Custo Total: {{ cheapestFreight.cost }}</p>
      <p>Tempo de Entrega: {{ cheapestFreight.deliveryTime }}</p>
    </div>
    <div v-if="fastestFreight">
      <h2>Frete Mais Rápido:</h2>
      <p>Transportadora: {{ fastestFreight.name }}</p>
      <p>Custo Total: {{ fastestFreight.cost }}</p>
      <p>Tempo de Entrega: {{ fastestFreight.deliveryTime }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      weight: 0,
      destination: "",
      cheapestFreight: null,
      fastestFreight: null,
    };
  },
  methods: {
    analyzeFreight() {
      // Fazer a requisição para a API REST e obter as cotações dos fretes
      fetch("http://localhost:3000/transport")
        .then((response) => response.json())
        .then((data) => {
          // Filtrar as cotações com base no peso e cidade de destino
          const filteredQuotes = data.filter(
            (quote) =>
              quote.city.toLowerCase() === this.destination.toLowerCase() &&
              ((this.weight <= 5 && quote.cost_transport_light) ||
                (this.weight > 5 && quote.cost_transport_heavy))
          );

          // Ordenar as cotações por custo e tempo de entrega
          filteredQuotes.sort((a, b) => {
            const aCost =
              this.weight <= 5
                ? parseFloat(a.cost_transport_light.substr(3))
                : parseFloat(a.cost_transport_heavy.substr(3));
            const bCost =
              this.weight <= 5
                ? parseFloat(b.cost_transport_light.substr(3))
                : parseFloat(b.cost_transport_heavy.substr(3));
            const aTime = parseFloat(a.lead_time);
            const bTime = parseFloat(b.lead_time);

            return aCost - bCost || aTime - bTime;
          });

          // Definir o frete mais econômico e o frete mais rápido
          this.cheapestFreight = filteredQuotes[0];
          this.fastestFreight = filteredQuotes[0];
        })
        .catch((error) => {
          console.error("Erro ao obter as cotações dos fretes:", error);
        });
    },
  },
};
</script>

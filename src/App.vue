<script setup lang="ts">
import { ref, onMounted } from "vue";

const API_URL = "scraperceb-production.up.railway.app";

const agio = ref<number>(0);
const carregando = ref(false);
const mensagem = ref("");
const erro = ref("");
const downloadUrl = ref("");

async function carregarAgio() {
  const resposta = await fetch(`${API_URL}/agio`);
  const dados = await resposta.json();
  agio.value = dados.agioPercentual;
}

async function gerarCatalogo() {
  carregando.value = true;
  mensagem.value = "";
  erro.value = "";
  downloadUrl.value = "";

  try {
    const atualizar = await fetch(`${API_URL}/agio`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        agioPercentual: agio.value
      })
    });

    if (!atualizar.ok) {
      throw new Error("Erro ao atualizar ágio.");
    }

    const resposta = await fetch(`${API_URL}/catalogo`, {
      method: "POST"
    });

    const dados = await resposta.json();

    if (!resposta.ok) {
      throw new Error(dados.erro || "Erro ao gerar catálogo.");
    }

    mensagem.value = `Catálogo gerado com ${dados.agioPercentual}% de ágio.`;
    downloadUrl.value = `${API_URL}${dados.download}`;
  } catch (e) {
    erro.value = e instanceof Error ? e.message : "Erro inesperado.";
  } finally {
    carregando.value = false;
  }
}

onMounted(carregarAgio);
</script>

<template>
  <main class="page">
    <header class="topbar">
      <div class="brand">
        <div class="logo-mark">C&B</div>
        <div>
          <h1>CAMARGO & BARROS</h1>
          <p>GERADOR DE CATÁLOGOS</p>
        </div>
      </div>
    </header>

    <section class="hero">
      <div class="panel">
        <span class="tag">FERRAMENTAS • VARIEDADES • ATACADO</span>

        <h2>Gerar catálogo PDF</h2>

        <p class="description">
          Escolha a porcentagem de ágio e gere um catálogo atualizado com preço final de venda.
        </p>

        <div class="form-group">
          <label>Ágio sobre o valor de atacado</label>

          <div class="input-wrap">
            <input
              v-model.number="agio"
              type="number"
              min="0"
              step="1"
              placeholder="0"
            />
            <span>%</span>
          </div>
        </div>

        <button class="primary" @click="gerarCatalogo" :disabled="carregando">
          {{ carregando ? "GERANDO CATÁLOGO..." : "GERAR CATÁLOGO" }}
        </button>

        <p v-if="mensagem" class="success">
          {{ mensagem }}
        </p>

        <p v-if="erro" class="error">
          {{ erro }}
        </p>

        <a
          v-if="downloadUrl"
          :href="downloadUrl"
          class="download"
          target="_blank"
        >
          BAIXAR CATÁLOGO PDF
        </a>
      </div>
    </section>
  </main>
</template>

<style scoped>
.page {
  min-height: 100vh;
  background: #f5f5f5;
  font-family: Arial, Helvetica, sans-serif;
  color: #222;
}

.topbar {
  background: #111;
  color: #fff;
  padding: 18px 24px;
  border-bottom: 5px solid #f6c400;
}

.brand {
  max-width: 980px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  gap: 14px;
}

.logo-mark {
  width: 58px;
  height: 58px;
  border-radius: 10px;
  background: #f6c400;
  color: #111;
  display: grid;
  place-items: center;
  font-weight: 900;
  font-size: 19px;
}

.brand h1 {
  margin: 0;
  font-size: 24px;
  letter-spacing: 1px;
}

.brand p {
  margin: 3px 0 0;
  font-size: 12px;
  color: #f6c400;
  font-weight: bold;
  letter-spacing: 1px;
}

.hero {
  max-width: 980px;
  margin: 44px auto;
  padding: 0 20px;
}

.panel {
  max-width: 480px;
  margin: 0 auto;
  background: #fff;
  border-radius: 10px;
  padding: 30px;
  border: 1px solid #ddd;
  box-shadow: 0 8px 22px rgba(0, 0, 0, 0.08);
}

.tag {
  display: inline-block;
  background: #f6c400;
  color: #111;
  font-size: 11px;
  font-weight: 900;
  padding: 7px 10px;
  border-radius: 4px;
  margin-bottom: 16px;
  letter-spacing: 0.5px;
}

h2 {
  margin: 0;
  font-size: 30px;
  text-transform: uppercase;
  color: #111;
}

.description {
  color: #555;
  font-size: 15px;
  line-height: 1.5;
  margin: 12px 0 24px;
}

.form-group label {
  display: block;
  font-weight: 800;
  text-transform: uppercase;
  font-size: 13px;
  margin-bottom: 8px;
}

.input-wrap {
  display: flex;
  align-items: center;
  border: 2px solid #111;
  border-radius: 8px;
  overflow: hidden;
  background: white;
  margin-bottom: 20px;
}

.input-wrap input {
  flex: 1;
  border: 0;
  padding: 15px;
  font-size: 22px;
  font-weight: bold;
  outline: none;
}

.input-wrap span {
  background: #111;
  color: #f6c400;
  padding: 16px 18px;
  font-size: 20px;
  font-weight: 900;
}

.primary {
  width: 100%;
  padding: 15px;
  background: #111;
  color: #f6c400;
  border: 0;
  border-radius: 8px;
  font-weight: 900;
  font-size: 15px;
  cursor: pointer;
  letter-spacing: 0.7px;
}

.primary:hover {
  background: #222;
}

.primary:disabled {
  background: #999;
  color: #eee;
  cursor: not-allowed;
}

.download {
  display: block;
  margin-top: 16px;
  text-align: center;
  background: #f6c400;
  color: #111;
  padding: 15px;
  border-radius: 8px;
  text-decoration: none;
  font-weight: 900;
}

.success {
  margin-top: 16px;
  padding: 12px;
  background: #ecfdf3;
  color: #166534;
  border: 1px solid #bbf7d0;
  border-radius: 8px;
  text-align: center;
  font-weight: bold;
}

.error {
  margin-top: 16px;
  padding: 12px;
  background: #fef2f2;
  color: #991b1b;
  border: 1px solid #fecaca;
  border-radius: 8px;
  text-align: center;
  font-weight: bold;
}
</style>
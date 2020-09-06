<template>
  <div id="app">
    <header class="header">
      <img src="./assets/images/techno.svg" alt="Techno" class="logo">
      <div class="carrinho-menu" @click="carrinhoAtivo = true">{{carrinhoTotal | formataPreco}} | {{carrinho.length}}</div>
    </header>

    <section class="produtos">
      <div v-for="produto in produtos" :key="produto.id" @click="abrirModal(produto.id)" class="produto">
        <img :src="produto.img" :alt="produto.name" class="produto-img">
        <div class="produto-info">
          <span class="produto-preco">{{produto.preco | formataPreco }}</span>
          <h2 class="produto-nome">{{produto.nome}}</h2>
        </div>
      </div>
    </section>

    <section class="modal" v-if="produto" @click="fecharModal">
      <div class="modal-container">
        <div class="modal-img">
          <img :src="produto.img" :alt="produto.nome">
        </div>
        <div class="modal-dados">
          <button @click="produto = false" class="modal-close">X</button>
          <span class="modal-preco">{{produto.preco | formataPreco}}</span>
          <h2 class="modal-nome">{{produto.nome}}</h2>
          <p>{{produto.descricao}}</p>
          <button v-if="produto.estoque > 0" class="modal-button" @click="adicionarItem">Adicionar Item</button>
          <button v-else class="modal-button esgotado" disabled>Produto Esgotado</button>
        </div>
        <div class="modal-reviews">
          <h2 class="modal-reviews-subtitulo">Avaliações</h2>
          <ul>
            <li v-for="(review, index) in produto.reviews" :key="review.nome + index" class="modal-review">
              <p class="modal-review-descricao">{{review.descricao}}</p>
              <p class="modal-review-usuario">{{review.nome}} | {{review.estrelas}} estrelas</p>
            </li>
          </ul>
        </div>
      </div>
    </section>

    <section class="modal-carrinho" :class="{ativo: carrinhoAtivo}" @click="fecharCarrinho">
      <div class="carrinho-container">
        <button @click="carrinhoAtivo = false" class="carrinho-fechar">X</button>
        <h2 class="carrinho-titulo">Carrinho</h2>
        <div v-if="carrinho.length > 0">
          <ul class="carrinho-lista">
            <li v-for="(item, index) in carrinho" :key="item.nome + index" class="carrinho-item">
              <p>{{item.nome}}</p>
              <p class="carrinho-item-preco">{{item.preco | formataPreco}}</p>
              <button class="carrinho-item-remover" @click="removerItem(index)">X</button>
            </li>
          </ul>
          <p class="carrinho-total">{{carrinhoTotal | formataPreco}}</p>
          <button class="carrinho-finalizar">Finalizar Compra</button>
        </div>
        <p v-else>O carrinho está vazio.</p>
      </div>
    </section>

    <div class="alerta" :class="{ativo: alertaAtivo}">
      <p class="alerta-mensagem">{{alertaMensagem}}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      produtos: [],
      produto: false,
      carrinho: [],
      alertaMensagem: null,
      alertaAtivo: false,
      carrinhoAtivo: false
    };
  },
  filters: {
    formataPreco(preco) {
      return preco.toLocaleString('pt-BR', {
        style: 'currency',
        currency: 'BRL'
      });
    }
  },
  computed: {
    carrinhoTotal() {
      let total = 0;

      if (this.carrinho.length) {
        this.carrinho.forEach(item => {
          total += item.preco;
        });
      }

      return total;
    }
  },
  methods: {
    fetchProdutos() {
      fetch('/api/produtos.json')
        .then(res => res.json())
        .then(data => {
          console.log(data);
          this.produtos = data;
        })
    },
    fetchProduto(id) {
      fetch(`/api/produtos/${id}/dados.json`)
        .then(res => res.json())
        .then(data => {
          this.produto = data;
        })
    },
    abrirModal(id) {
      this.fetchProduto(id);

      window.scrollTo({
        top: 0,
        behavior: 'smooth'
      });
    },
    fecharModal({target, currentTarget}) {
      if (target === currentTarget) {
        this.produto = false;
      }
    },
    fecharCarrinho({target, currentTarget}) {
      if (target === currentTarget) {
        this.carrinhoAtivo = false;
      }
    },
    adicionarItem() {
      this.produto.estoque--;

      const { id, nome, preco } = this.produto;

      this.carrinho.push({ id, nome, preco });

      this.alerta(`${nome} adicionado ao carrinho.`);
    },
    removerItem(index) {
      this.carrinho.splice(index, 1);
    },
    checarLocalStorage() {
      if (window.localStorage.carrinho) {
        this.carrinho = JSON.parse(window.localStorage.carrinho);
      }
    },
    compararEstoque() {
      const items = this.carrinho.filter(({ id }) => id === this.produto.id);

      this.produto.estoque -= items.length;
    },
    alerta(mensagem) {
      this.alertaMensagem = mensagem;
      this.alertaAtivo = true;

      setTimeout(() => {
        this.alertaAtivo = false;
      }, 1500);
    },
    router() {
      const hash = document.location.hash.replace('#', '');

      if (hash) {
        this.fetchProduto(hash);
      }
    }
  },
  watch: {
    produto() {
      document.title = this.produto.nome || 'Loja';

      const hash = this.produto.id || '';
      history.pushState(null, null, `#${hash}`);

      if (this.produto) {
        this.compararEstoque();
      }
    },
    carrinho() {
      window.localStorage.carrinho = JSON.stringify(this.carrinho);
    }
  },
  created() {
    this.fetchProdutos();
    this.router();
    this.checarLocalStorage();
  }
}
</script>

<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  ul {
    list-style: none;
  }

  body {
    background: linear-gradient(to right, #1a1a1a 30%, #ffffff 30%);
    font-family: 'Noto Serif', serif;
  }

  #app {
    padding: 0 80px;
  }

  .header {
    display: flex;
    justify-content: space-between;
    max-width: 900px;
    margin: 0 auto;
    padding: 20px 0;
  }

  .logo {
    width: 80px;
  }

  .carrinho-menu::after {
    content: '';
    display: inline-block;
    background: url('./assets/images/carrinho.svg') no-repeat center center;
    width: 25px;
    height: 25px;
    margin-left: 10px;
  }
  .carrinho-menu {
    display: flex;
    align-items: center;
    cursor: pointer;
  }

  .produtos {
    max-width: 900px;
    margin: 100px auto 0 auto;
  }

  .produto {
    display: flex;
    align-items: center;
    margin-bottom: 40px;
    background-color: #ffffff;
    box-shadow: 0 0 2rem rgba(0, 0, 0, .1);
    cursor: pointer;
  }

  .produto-img {
    max-width: 300px;
    margin-right: 40px;
  }

  .produto-preco {
    color: rgba(0, 0, 0, .5);
  }

  .produto-nome {
    font-size: 3rem;
    line-height: 1;
  }

  .modal::before {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;
    background-color: rgba(0, 0, 0, .5);
  }
  .modal {
    display: flex;
    flex-direction: column;
    align-items: center;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    padding: 80px;
  }

  .modal-container {
    position: relative;
    z-index: 1;
    background: linear-gradient(to right, transparent 250px, #ffffff 250px);
    display: grid;
    align-items: end;
    grid-gap: 50px;
    padding: 50px 50px 50px 0;
    animation: fadeInRight .3s forwards;
  }

  .modal-close {
    border-radius: 50%;
    border: 2px solid #000000;
    background-color: #ffffff;
    width: 40px;
    height: 40px;
    position: absolute;
    top: -10px;
    right: -10px;
    font-size: 1rem;
    box-shadow: 0 3px 4px rgba(0, 0, 0, .1), 0 4px 10px rgba(0, 0, 0, .2);
    cursor: pointer;
  }

  .modal-img {
    grid-column: 1;
    box-shadow: 0 3px 4px rgba(0, 0, 0, .1), 0 4px 10px rgba(0, 0, 0, .2);
  }
  .modal-img img {
    max-width: 300px;
    display: block;
  }

  .modal-dados {
    grid-column: 2;
    max-width: 600px;
  }

  .modal-nome {
    font-size: 3rem;
  }

  .modal-button {
    margin-top: 80px;
    background-color: #000000;
    color: #ffffff;
    cursor: pointer;
    border: none;
    font-size: 1rem;
    padding: 10px 25px;
    font-family: 'Noto Serif', serif;
  }
  .modal-button:active {
    background-color: #808080;
  }
  .modal-button.esgotado {
    background-color: #808080;
    cursor: default;
  }

  .modal-reviews {
    grid-column: 2;
  }

  .modal-reviews-subtitulo {
    font-size: 1.75rem;
  }

  .modal-review {
    border-bottom: 1px solid rgba(0, 0, 0, .1);
    padding-bottom: 20px;
  }

  .modal-review-descricao {
    color: rgba(0, 0, 0, .7);
    margin: 20px 0 5px 0;
  }

  .modal-review-usuario {
    font-weight: bold;
  }

  .modal-carrinho::before {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;
    background-color: rgba(0, 0, 0, .5);
  }
  .modal-carrinho {
    display: flex;
    flex-direction: column;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    padding: 20px;
    display: none;
  }
  .modal-carrinho.ativo {
    display: block;
  }

  .carrinho-fechar {
    border-radius: 50%;
    border: 2px solid #000000;
    background-color: #ffffff;
    width: 40px;
    height: 40px;
    position: absolute;
    top: -10px;
    right: -10px;
    font-size: 1rem;
    box-shadow: 0 3px 4px rgba(0, 0, 0, .1), 0 4px 10px rgba(0, 0, 0, .2);
    cursor: pointer;
  }

  .carrinho-container {
    position: relative;
    margin: 80px auto;
    background: #ffffff;
    padding: 40px;
    max-width: 800px;
    animation: fadeInDow .3s forwards;
  }

  .carrinho-titulo {
    margin-bottom: 10px;
    padding-bottom: 20px;
    border-bottom: 2px solid #000000;
  }

  .carrinho-item {
    display: grid;
    grid-template-columns: 1fr 1fr 50px;
    border-bottom: 1px solid rgba(0, 0, 0, .1);
    padding: 10px 0;
  }

  .carrinho-item-preco {
    text-align: right;
  }

  .carrinho-item-remover {
    border: none;
    font-size: 1rem;
    background: transparent;
    cursor: pointer;
  }

  .carrinho-total {
    text-align: right;
    padding: 10px 50px 10px 0;
    margin-bottom: 20px;
    border-bottom: 2px solid #000000;
  }

  .carrinho-finalizar {
    display: block;
    margin-left: auto;
    background: #000000;
    color: #ffffff;
    cursor: pointer;
    font-size: 1rem;
    padding: 10px 25px;
    border: none;
    font-family: 'Noto Serif', serif;
  }

  .alerta {
    position: absolute;
    top: 20px;
    left: 0;
    z-index: 10;
    width: 100%;
    text-align: center;
    display: none;
  }
  .alerta.ativo {
    display: block;
    animation: fadeInDow .3s forwards;
  }

  .alerta-mensagem {
    background: #ffffff;
    display: inline-block;
    padding: 20px 40px;
    border: 2px solid #000000;
    box-shadow: 0 3px 4px rgba(0, 0, 0, .1), 0 4px 10px rgba(0, 0, 0, .2);
  }

  @media screen and (max-width: 795px) {
    #app {
      padding: 0 10px;
    }

    .produtos {
      margin-top: 40px;
    }
    .produto {
      flex-direction: column;
      align-items: flex-start;
      max-width: 300px;
      margin: 30px auto;
    }
    .produto-info {
      padding: 20px;
    }
    .produto-img {
      max-width: 100%;
    }
    .produto-nome {
      font-size: 1.5rem;
    }

    .modal {
      padding: 10px;
    }
    .modal-container {
      grid-gap: 20px;
      background: #ffffff;
      padding: 10px 0;
    }
    .modal-img {
      grid-row: 2;
    }
    .modal-img img {
      width: 100%;
      max-width: initial;
    }
    .modal-dados {
      grid-column: 1;
      padding: 10px;
    }
    .modal-button {
      margin-top: 20px;
    }
    .modal-reviews {
      grid-column: 1;
      padding: 10px;
    }

    .modal-carrinho {
      padding: 10px;
    }
  }

  @keyframes fadeInRight {
    from {
      opacity: 0;
      transform: translate3d(50px, 0, 0);
    }
    to {
      opacity: 1;
      transform: translate3d(0, 0, 0);
    }
  }

  @keyframes fadeInDow {
    from {
      opacity: 0;
      transform: translate3d(0, -30px, 0);
    }
    to {
      opacity: 1;
      transform: translate3d(0, 0, 0);
    }
  }
</style>

<template>
  <div class="container-fluid my-4">
    <div class="row justify-content-center">
      <img src="../src/assets/logo-butcher.svg" height="80px" class="text-center mb-5 align-itens-center d-flex" alt="Logo Butcger" />
    </div>
    <div class="row">
      <div class="col-lg-12 mb-4">
        <h3 class="text-light mb-4">Escolha seus Produtos</h3>
        <div class="row">
          <div class="col-6 col-lg-3 mb-3" v-for="item in menuItems" :key="item.id">
            <div class="card shadow-sm h-100">
              <img :src="item.imagem" class="card-img-top" alt="Imagem do Burger" style="height: 180px; object-fit: cover;">
              <div class="card-body d-flex flex-column">
                <h5 class="card-title">{{ item.nome }}</h5>
                <h6 class="card-text">{{ item.descricao }}  </h6>
                <p class="card-price text-muted">R$ {{ item.preco.toFixed(2).replace('.', ',') }}</p>
                <button @click="addToCart(item)" class="btn btn-primary mt-auto">
                  Adicionar no Carrinho
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="col-lg-12">
        <h3 class="text-light mb-4 mt-4">Seu pedido</h3>
        <div class="sticky-top" style="top: 20px;">
          <div class="card p-4 shadow-sm border-0">
            <ul class="list-group list-group-flush mb-3" v-if="cart.length > 0">
              <li class="list-group-item d-flex justify-content-between align-items-center py-2"
                  v-for="item in cart" :key="item.id">
                
                <span>{{ item.nome }} x{{ item.quantidade }}</span>
                
                <div class="btn-group btn-group-sm" role="group">
                  <button @click="changeQuantity(item.id, -1)" class="btn btn-outline-danger px-3">-</button>
                  <span class="d-flex align-items-center px-2">R$ {{ (item.preco * item.quantidade).toFixed(2).replace('.', ',') }}</span>
                  <button @click="changeQuantity(item.id, 1)" class="btn btn-outline-success px-3">+</button>
                </div>
              </li>
            </ul>
            
            <p v-else class="text-center text-muted py-3">O carrinho está vazio. Adicione itens!</p>

            <hr>
            
            <h4 class="mb-3">Total: <strong>R$ {{ totalValue.toFixed(2).replace('.', ',') }}</strong></h4>
            
            <div class="alert alert-success text-center border-0 shadow-sm" v-if="cart.length > 0">
                <p class="mb-1 fw-bold">Pague via PIX (Valor de R$ {{ totalValue.toFixed(2).replace('.', ',') }})</p>
                <p class="mb-1 small mt-3">Chave Pix:</p>
                
                <code class="d-block p-2 bg-white border rounded text-dark fs-6">
                    {{ staticPixKey }}
                </code>
                
                <button @click="copyPixKey" class="btn btn-sm btn-secondary py-2 px-3 mt-3">
                    <i class="bi bi-clipboard"></i> {{ copied ? ' Copiado!' : 'Copiar Chave' }}
                </button>
            <p class="mt-3 mb-0 small text-center text-muted">A chave Pix é fixa. O valor deve ser inserido manualmente no app do seu banco.</p>

            </div>
            
            <a :href="whatsappLink" target="_blank" 
               class="btn btn-success btn-lg mt-3" 
               :class="{ 'disabled': cart.length === 0 }">
              <i class="bi bi-whatsapp"></i> Enviar Pedido e Comprovante
            </a>
            
            <p class="mt-3 small text-center text-muted"><strong>INSTRUÇÃO:</strong> Por favor, envie seu comprovante Pix junto com este pedido.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // ⚠️ Substitua pelos seus dados reais!
      whatsappNumber: '5511999999999', // Seu número (55 + DDD + Número)
      staticPixKey: '12345678900@exemplo.com.br', // SUA CHAVE PIX REAL AQUI
      
      copied: false,
      cart: [],
      menuItems: [
        { id: 1, nome: "Humildão", descricao:"Pão, Carne 160g, Queijo e Maionese" ,preco: 25.00, imagem: '/img/humildao.png' },
        { id: 2, nome: "Sou Fit", descricao:"Pão, Carne 160g, Queijo, Alface, Tomate, Picles e Maionese" ,preco: 12.00, imagem: '/img/soufit.png' },
        { id: 3, nome: "Peppa", descricao:"Pão, Carne 160g, Queijo, Bacon e Maionese" , preco: 7.00, imagem: '/img/peppa.png' },
        { id: 4, nome: "Chimichurri", descricao:"Pão, Carne 220g, Queijo, Molho Chimichurri" ,preco: 18.00, imagem: '/img/chimi.png' },
        { id: 5, nome: "Coca-cola Lata", preco: 18.00, imagem: '/img/coca.png' },
        { id: 6, nome: "Coca-cola s/açucar Lata", preco: 18.00, imagem: '/img/coca-sacucar.png' },
        { id: 7, nome: "Chopp Pilsen 1L", preco: 18.00, imagem: '/img/chopp.png' },
        { id: 8, nome: "Chopp IPA 1L", preco: 18.00, imagem: '/img/chopp.png' },
        { id: 9, nome: "Água c/ gás", preco: 18.00, imagem: '/img/agua-gas.png' },
        { id: 10, nome: "Água s/ gás", preco: 18.00, imagem: '/img/agua.png' },
      ],
    };
  },
  computed: {
    totalValue() {
      return this.cart.reduce((sum, item) => sum + (item.preco * item.quantidade), 0);
    },
    
    whatsappMessage() {
      if (this.cart.length === 0) return '';
      
      let message = "--- Reserva de Burger ---\n\n";
      
      // Lista de Itens
      this.cart.forEach(item => {
        message += `* ${item.nome} (x${item.quantidade}): R$ ${(item.preco * item.quantidade).toFixed(2).replace('.', ',')}\n`;
      });

      // Total e Instruções Pix
      message += `\nVALOR FINAL: R$ ${this.totalValue.toFixed(2).replace('.', ',')}\n`;
      
      return encodeURIComponent(message);
    },
    
    whatsappLink() {
      return `https://wa.me/${this.whatsappNumber}?text=${this.whatsappMessage}`;
    }
  },
  methods: {
    // --- Lógica do Carrinho ---
    addToCart(item) {
      this.changeQuantity(item.id, 1);
    },

    changeQuantity(itemId, delta) {
      const existingIndex = this.cart.findIndex(i => i.id === itemId);
      const menuItem = this.menuItems.find(i => i.id === itemId);

      if (existingIndex !== -1) {
        const newQuantity = this.cart[existingIndex].quantidade + delta;
        
        if (newQuantity <= 0) {
          this.cart.splice(existingIndex, 1);
        } else {
          this.cart[existingIndex].quantidade = newQuantity;
        }
      } else if (delta > 0 && menuItem) {
        this.cart.push({ ...menuItem, quantidade: 1 });
      }
    },

    // --- Lógica de Copiar Pix ---
    async copyPixKey() {
      if (this.copied) return;
      try {
        await navigator.clipboard.writeText(this.staticPixKey);
        this.copied = true;
        setTimeout(() => {
          this.copied = false;
        }, 2000);
      } catch (err) {
        alert('Erro ao copiar! Por favor, copie manualmente: ' + this.staticPixKey);
      }
    }
  }
};
</script>

<style>
.btn-primary, .btn-success {
  background-color: #5dd62c !important;
  border: 1px solid #5dd62c !important;
  color: #245510 !important;
  font-weight: 500 !important;
}
.btn-primary:hover, .btn-success:hover {
  background-color: #4aad21 !important;
  border: 1px solid #4aad21 !important;
}
.btn-secondary {
  background-color: #235a0c !important;
  color: #fff;
}
.btn-secondary:hover {
  background-color: #163e06 !important;
  color: #fff;
}
.card-title{
  font-weight: 700;
  color: #202020 !important;
}
.card-text {
  font-size: 14px !important;
  color: #7a7a7a;
}

.card-price {
  font-size: 16px;
  font-weight: 700;
  color: #202020;
  margin-top: auto;
}
.alert-success {
  background-color: #cbfec2 !important;
  color: #163e06 !important;
}
</style>
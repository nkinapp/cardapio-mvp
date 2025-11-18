<template>
  <div class="container-fluid my-5">
    <h1 class="text-center mb-5 fw-bold text-primary">🍽️ Cardápio Digital MVP</h1>
    
    <div class="row">
      <div class="col-lg-12 mb-4">
        <h2>Escolha seus Produtos</h2>
        <div class="row">
          <div class="col-6 col-lg-4 mb-3" v-for="item in menuItems" :key="item.id">
            <div class="card shadow-sm h-100">
              <img :src="item.imagem" class="card-img-top" alt="Imagem do produto" style="height: 150px; object-fit: cover;">
              <div class="card-body d-flex flex-column">
                <h5 class="card-title">{{ item.nome }}</h5>
                <p class="card-text text-muted">R$ {{ item.preco.toFixed(2).replace('.', ',') }}</p>
                <button @click="addToCart(item)" class="btn btn-primary mt-auto">
                  Adicionar
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="col-lg-12">
        <div class="sticky-top" style="top: 20px;">
          <div class="card p-4 shadow-sm border-0">
            <h3>🛒 Seu Pedido</h3>

            <ul class="list-group list-group-flush mb-3" v-if="cart.length > 0">
              <li class="list-group-item d-flex justify-content-between align-items-center py-2"
                  v-for="item in cart" :key="item.id">
                
                <span>{{ item.nome }} x{{ item.quantidade }}</span>
                
                <div class="btn-group btn-group-sm" role="group">
                  <button @click="changeQuantity(item.id, -1)" class="btn btn-outline-danger px-2">-</button>
                  <span class="btn btn-light px-2">R$ {{ (item.preco * item.quantidade).toFixed(2).replace('.', ',') }}</span>
                  <button @click="changeQuantity(item.id, 1)" class="btn btn-outline-success px-2">+</button>
                </div>
              </li>
            </ul>
            
            <p v-else class="text-center text-muted py-3">O carrinho está vazio. Adicione itens!</p>

            <hr>
            
            <h4 class="mb-3">Total: **R$ {{ totalValue.toFixed(2).replace('.', ',') }}**</h4>
            
            <div class="alert alert-success text-center border-0 shadow-sm" v-if="cart.length > 0">
                <p class="mb-1 fw-bold">Pague via PIX (Valor de R$ {{ totalValue.toFixed(2).replace('.', ',') }})</p>
                <p class="mb-1 small">Chave Pix:</p>
                
                <code class="d-block p-2 bg-white border rounded text-dark fs-6">
                    {{ staticPixKey }}
                </code>
                
                <button @click="copyPixKey" class="btn btn-sm btn-outline-success mt-2">
                    <i class="bi bi-clipboard"></i> {{ copied ? '✅ Copiado!' : 'Copiar Chave' }}
                </button>
            </div>
            
            <a :href="whatsappLink" target="_blank" 
               class="btn btn-success btn-lg mt-3" 
               :class="{ 'disabled': cart.length === 0 }">
              <i class="bi bi-whatsapp"></i> Enviar Pedido e Comprovante
            </a>
            
            <p class="mt-2 small text-center text-muted">A chave Pix é fixa. O valor deve ser inserido manualmente no app do seu banco.</p>
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
        { id: 1, nome: "Hamburguer Clássico", preco: 25.00, imagem: 'https://via.placeholder.com/400x150/FF5733/FFFFFF?text=Hamburguer' },
        { id: 2, nome: "Batata Frita G", preco: 12.00, imagem: 'https://via.placeholder.com/400x150/FFC300/FFFFFF?text=Batata' },
        { id: 3, nome: "Refrigerante Lata", preco: 7.00, imagem: 'https://via.placeholder.com/400x150/DAF7A6/FFFFFF?text=Refri' },
        { id: 4, nome: "Milkshake Chocolate", preco: 18.00, imagem: 'https://via.placeholder.com/400x150/C70039/FFFFFF?text=Milkshake' },
      ],
    };
  },
  computed: {
    totalValue() {
      return this.cart.reduce((sum, item) => sum + (item.preco * item.quantidade), 0);
    },
    
    whatsappMessage() {
      if (this.cart.length === 0) return '';
      
      let message = "*--- PEDIDO ONLINE ---\n\n*";
      
      // Lista de Itens
      this.cart.forEach(item => {
        message += `* ${item.nome} (x${item.quantidade}): R$ ${(item.preco * item.quantidade).toFixed(2).replace('.', ',')}\n`;
      });

      // Total e Instruções Pix
      message += `\n*VALOR FINAL:* R$ ${this.totalValue.toFixed(2).replace('.', ',')}\n`;
      message += `*CHAVE PIX:* ${this.staticPixKey}\n\n`;
      message += "✅ *INSTRUÇÃO*: Por favor, envie seu comprovante Pix junto com este pedido, e informe seu nome e endereço completo para entrega.";

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
/* Estilos globais se necessários. Você pode usar o VUE CLI para adicionar estilos */
</style>
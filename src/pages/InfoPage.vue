<template>
  <q-page class="q-pa-md q-pb-xl">
    <!-- Developer Card -->
    <q-card flat bordered class="q-mb-lg rounded-borders">
      <q-card-section class="row items-center q-pb-sm">
        <q-icon name="person" color="primary" size="sm" class="q-mr-sm" />
        <div class="text-h6">Desenvolvedora</div>
      </q-card-section>
      <q-separator />
      <q-card-section class="q-pt-md">
        <div class="row items-center q-gutter-md">
          <q-avatar color="primary" text-color="white" size="56px">
            <q-icon name="person" size="md" />
          </q-avatar>
          <div>
            <div class="text-body1 text-weight-bold">Cristiane da Rocha</div>
            <q-btn
              flat
              dense
              no-caps
              icon="code"
              label="Ver repositório"
              color="primary"
              class="q-pl-none q-mt-xs"
              href="https://github.com/crisrochadev/lista-de-compras"
              target="_blank"
            />
          </div>
        </div>
      </q-card-section>
    </q-card>

    <!-- Usage Guide -->
    <q-card flat bordered class="rounded-borders">
      <q-card-section class="row items-center q-pb-sm">
        <q-icon name="lightbulb" color="warning" size="sm" class="q-mr-sm" />
        <div class="text-h6">Como usar</div>
      </q-card-section>
      <q-separator />
      <q-card-section>
        <p class="text-body2 text-grey-7 q-mb-md">
          A Lista de Compras é um organizador simples e prático para nunca mais esquecer nada no mercado.
          Veja como tirar o melhor proveito:
        </p>

        <q-timeline color="primary" layout="dense">
          <q-timeline-entry
            v-for="step in steps"
            :key="step.title"
            :icon="step.icon"
            :color="step.color"
          >
            <template #title>
              <span class="text-body1 text-weight-bold">{{ step.title }}</span>
            </template>
            <span class="text-body2 text-grey-7">{{ step.description }}</span>
          </q-timeline-entry>
        </q-timeline>

        <q-banner rounded class="bg-primary text-white q-mt-md">
          <template #avatar>
            <q-icon name="tips_and_updates" />
          </template>
          <strong>Dica:</strong> Ao terminar as compras, use o menu de contexto (clique longo ou botão direito)
          para arquivar os itens. Você pode desarquivá-los a qualquer momento para reutilizá-los na próxima compra.
        </q-banner>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
const steps = [
  {
    icon: 'add_circle',
    color: 'primary',
    title: '1. Crie sua lista base',
    description: 'Adicione todos os itens que você normalmente precisa ter na despensa: arroz, feijão, azeite, sabão em pó, shampoo... tudo que faz parte da sua rotina de compras.'
  },
  {
    icon: 'search',
    color: 'info',
    title: '2. Verifique o que tem em casa',
    description: 'Antes de ir ao mercado, vá até a dispensa, geladeira e armários. Confira cada item da lista com o que realmente está em casa.'
  },
  {
    icon: 'check_circle',
    color: 'positive',
    title: '3. Marque o que já possui',
    description: 'Para os itens que já estão em quantidade suficiente, marque o checkbox. Eles ficarão riscados na lista, indicando que não precisam ser comprados.'
  },
  {
    icon: 'radio_button_unchecked',
    color: 'warning',
    title: '4. Desmarque o que está faltando',
    description: 'Se um item acabou ou está quase no fim, desmarque-o para que apareça normalmente na lista como item a ser comprado.'
  },
  {
    icon: 'tune',
    color: 'secondary',
    title: '5. Ajuste as quantidades',
    description: 'Use os botões ‹ e › ao lado de cada item para definir exatamente quantas unidades, pacotes ou caixas você precisa comprar.'
  },
  {
    icon: 'shopping_cart',
    color: 'primary',
    title: '6. Vá às compras!',
    description: 'No mercado, conforme você coloca os itens no carrinho, marque-os na lista. Ao final, tudo estará marcado e você não esquecerá nada!'
  }
]
</script>

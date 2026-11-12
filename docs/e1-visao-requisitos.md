# E1 — Visão e requisitos do EntregaJá

## Visão e problema

O EntregaJá é uma plataforma web de e-commerce com entrega para conectar clientes, lojistas e entregadores de um marketplace de bairro. Substitui o atendimento manual por mensagens, reduz perda de pedidos e venda sem estoque e mantém a operação de catálogo disponível durante picos e falhas de pagamento. A arquitetura parte de um monólito modular em camadas, preparado para evolução gradual em serviços.

## Stakeholders e atores

| Ator | Papel |
|---|---|
| Gestores do marketplace | Escalabilidade, estabilidade em promoções e evolução sustentável |
| Cliente | Busca e compra produtos, acompanha pedidos e avalia após a entrega |
| Lojista / administrador de catálogo | Mantém produtos, categorias, preços e estoque |
| Entregador | Consulta pedidos atribuídos e atualiza status da entrega |
| Administrador | Gerencia usuários, acompanha pedidos e relatórios |
| Gateway de pagamento | Sistema externo simulado que autoriza e confirma cobranças |

## Requisitos funcionais

| ID | Requisito |
|---|---|
| RF01 | Manter catálogo de produtos e categorias, com nome, preço, categoria e estoque inicial. |
| RF02 | Buscar e filtrar produtos por nome/categoria com resultado paginado. |
| RF03 | Adicionar/remover itens no carrinho e recalcular subtotal automaticamente. |
| RF04 | Calcular o total do pedido: itens mais frete. |
| RF05 | Realizar checkout e criar pedido com status inicial CRIADO. |
| RF06 | Reservar e baixar estoque dos itens na confirmação do pedido. |
| RF07 | Processar pagamento com idempotência. |
| RF08 | Atualizar status do pedido e da entrega ao longo do ciclo de vida. |
| RF09 | Consultar status atual e histórico de alterações do pedido. |
| RF10 | Avaliar produto/pedido após a conclusão da entrega. |
| RF11 | Autenticar usuários e aplicar controle de acesso por perfil. |
| RF12 | Permitir ao lojista repor o estoque disponível. |

## Requisitos não funcionais

| ID | Atributo | Requisito |
|---|---|---|
| RNF01 | Escalabilidade | Suportar 10× o acesso normal ao catálogo em promoções, com p95 < 2 s. |
| RNF02 | Desempenho | Busca de produtos com p95 < 1,5 s em uso normal. |
| RNF03 | Consistência | Em compras concorrentes da última unidade, impedir saldo negativo. |
| RNF04 | Disponibilidade | Catálogo e carrinho continuam operacionais durante indisponibilidade do gateway. |
| RNF05 | Resiliência | Circuit breaker em pagamento lento/com falha; pedido permanece PENDENTE_PAGAMENTO. |
| RNF06 | Segurança | Proteger dados do cliente e pagamento; não armazenar dados sensíveis de cartão. |
| RNF07 | Integrabilidade | Permitir substituir o gateway por adaptador sem alterar o núcleo do domínio. |

## Regras de negócio

- **RN01:** estoque nunca fica negativo; o pedido só confirma se há estoque para todos os itens.
- **RN02:** pedido só avança para EM_PREPARO depois da confirmação do pagamento (PAGO).
- **RN03:** total é a soma de preço × quantidade dos itens mais o frete vigente no checkout.
- **RN04:** toda cobrança usa chave de idempotência para evitar duplicidade em reenvios.
- **RN05:** avaliação só é permitida em pedido ENTREGUE.
- **RN06:** transições seguem ordem estrita e não retrocedem, exceto cancelamento explícito.
- **RN07:** cliente acessa apenas seus pedidos; lojista não acessa dados financeiros do cliente.

## Drivers arquiteturais

1. Consistência de estoque sem prejudicar a leitura escalável do catálogo (RN01, RNF01, RNF03).
2. Resiliência do pagamento e isolamento de falhas para preservar catálogo e carrinho (RN04, RNF04, RNF05).
3. Modificabilidade para substituir integrações e, quando justificado, extrair módulos como serviços (RNF07).

## Estilo inicial

Monólito modular em camadas: apresentação (controllers REST/DTOs), aplicação (casos de uso/transações), domínio (entidades e regras) e persistência (repositórios Spring Data JPA). A escolha reduz custo operacional inicial, mantém fronteiras por contexto de negócio e viabiliza futura extração de serviços com interfaces e adaptadores.

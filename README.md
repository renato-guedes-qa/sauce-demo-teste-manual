# Projeto — SauceDemo

Documentação de testes funcionais manuais para o [SauceDemo](https://www.saucedemo.com/), aplicação de e-commerce mantida pela Sauce Labs para prática e treinamento de QA. O projeto cobre os dois fluxos de maior risco da aplicação: autenticação e finalização de compra.

## Sumário
- [Escopo e Regras de Negócio](#escopo-e-regras-de-negócio)
- [Itens Fora do Escopo](#itens-fora-do-escopo)
- [Plano de Teste](#plano-de-teste)
- [Casos de Teste](#casos-de-teste)
- [Conclusão](#conclusão)
- [Incident Reports](./bugs.md)

## Escopo e Regras de Negócio

### Autenticação
- O acesso é feito com usuários pré-cadastrados pela própria aplicação (não há cadastro de novos usuários).
- Credenciais inválidas devem impedir o acesso e exibir mensagem de erro.
- O usuário `locked_out_user` deve ser bloqueado mesmo com senha correta, com mensagem específica.
- Login bem-sucedido redireciona para a página de produtos.
- Logout deve encerrar a sessão e retornar à tela de login.

### Checkout
- É necessário ao menos um produto no carrinho para iniciar o checkout.
- A etapa "Your Information" exige nome, sobrenome e CEP — todos obrigatórios.
- Campos obrigatórios em branco devem impedir o avanço, com mensagem de erro.
- O resumo do pedido (Overview) deve refletir corretamente os itens e valores selecionados.
- O usuário pode cancelar o checkout a qualquer etapa e retornar ao carrinho/produtos.
- A finalização exibe mensagem de confirmação do pedido.
- **Comportamento conhecido:** o usuário `problem_user` apresenta múltiplas falhas na vitrine, no carrinho e no checkout — bugs documentados com evidência (ver TC-015 a TC-015b), não regras de negócio esperadas.
- **Comportamento conhecido:** mesmo com `standard_user`, o checkout é concluído com carrinho vazio, gerando pedido com 0 itens e $0.00 — violação da regra acima, documentada com evidência (ver [TC-016](#casos-de-teste)).

## Itens Fora do Escopo

Estas áreas existem na aplicação, mas não fazem parte desta rodada de testes — o escopo foi restrito às funcionalidades de maior risco (controle de acesso e conclusão de transação):

- Ordenação e filtro do catálogo de produtos.
- Comportamento do contador do carrinho fora do fluxo de checkout.
- "Reset App State" (menu).
- Link externo "About" (redireciona para fora da aplicação).
- Comportamentos específicos de outros usuários especiais (`error_user`, `performance_glitch_user`, `visual_user`), exceto o caso documentado do `problem_user` no checkout.

## Plano de Teste

**Objetivo:** validar os fluxos de autenticação e finalização de compra da aplicação SauceDemo, priorizando os cenários de maior impacto em caso de falha.

**Tipo de teste:** Funcional Manual

**Cobertura:**
- Autenticação: login válido, login inválido, login bloqueado, logout.
- Checkout: pré-condição de carrinho, obrigatoriedade de campos, cancelamento, resumo do pedido, finalização, regressão com `problem_user`.

**Critério de classificação de divergências:** nem toda diferença entre esperado e observado tem a mesma natureza — cada uma foi tratada de forma distinta:
- **Defeito** (viola regra de negócio) → registrado como incidente em [`bugs.md`](./bugs.md), com severidade e prioridade avaliadas separadamente.
- **Requisito impreciso** (regra deduzida sem acesso à especificação original da aplicação) → corrigido diretamente no caso de teste.
- **Observação de usabilidade** (comportamento correto, mas questionável do ponto de vista de experiência) → registrada como nota no caso de teste, sem virar incidente.

## Casos de Teste

### Autenticação

**TC-001 — Login com credenciais válidas**
Pré-condição: nenhuma.
Passos: acessar a tela de login → informar `standard_user` / `secret_sauce` → confirmar.
Resultado esperado: usuário autenticado e redirecionado para a página de produtos.

**TC-002 — Login com senha inválida**
Passos: informar usuário válido e senha incorreta → confirmar.
Resultado esperado: acesso negado, com mensagem de erro.

**TC-003 — Login com usuário inexistente**
Passos: informar um nome de usuário não cadastrado → confirmar.
Resultado esperado: acesso negado, com mensagem de erro.

**TC-004 — Login sem informar usuário**
Passos: deixar o campo usuário em branco → confirmar.
Resultado esperado: sistema informa que o campo é obrigatório.

**TC-005 — Login sem informar senha**
Passos: deixar o campo senha em branco → confirmar.
Resultado esperado: sistema informa que o campo é obrigatório.

**TC-006 — Login com usuário bloqueado**
Passos: informar `locked_out_user` / `secret_sauce` → confirmar.
Resultado esperado: acesso negado, com mensagem específica de bloqueio.

**TC-007 — Logout**
Pré-condição: usuário autenticado.
Passos: acessar o menu → selecionar Logout.
Resultado esperado: sessão encerrada, retorno à tela de login.

### Checkout

**TC-008 — Iniciar checkout com item no carrinho**
Pré-condição: usuário autenticado; produto adicionado ao carrinho (passo de preparo, não regra de negócio testada).
Passos: acessar o carrinho → clicar em Checkout.
Resultado esperado: usuário direcionado à etapa "Your Information".

**TC-009 — Avançar checkout sem informar nome**
Passos: deixar o campo nome em branco → preencher os demais → continuar.
Resultado esperado: sistema impede o avanço e informa o campo obrigatório.

**TC-010 — Avançar checkout sem informar sobrenome**
Passos: deixar o campo sobrenome em branco → preencher os demais → continuar.
Resultado esperado: sistema impede o avanço e informa o campo obrigatório.

**TC-011 — Avançar checkout sem informar CEP**
Passos: deixar o campo CEP em branco → preencher os demais → continuar.
Resultado esperado: sistema impede o avanço e informa o campo obrigatório.

**TC-012 — Verificar resumo do pedido**
Pré-condição: informações do checkout preenchidas corretamente.
Passos: avançar para a etapa Overview → conferir itens, quantidades e valores.
Resultado esperado: dados exibidos correspondem aos produtos selecionados.

**TC-013 — Cancelar checkout na etapa "Your Information"**
Passos: iniciar o checkout → na etapa "Your Information", clicar em Cancel sem avançar.
Resultado esperado: usuário retorna ao carrinho.
Observação: consistente com a jornada — é o passo imediatamente anterior.

**TC-013a — Cancelar checkout na etapa Overview (revisão final)**
Passos: iniciar o checkout → preencher informações → avançar até Overview → clicar em Cancel.
Resultado esperado: usuário retorna à vitrine (página de produtos).
Observação (usabilidade): não há regra de negócio que determine este destino — é decisão de design, não falha funcional. Do ponto de vista de jornada, cancelar após já ter revisado o pedido poderia igualmente justificar retorno ao carrinho, permitindo nova checagem dos itens sem exigir navegação adicional pelo catálogo. Vale como ponto de discussão de usabilidade, não como incidente.

**TC-014 — Finalizar compra com dados válidos**
Pré-condição: checkout preenchido corretamente.
Passos: confirmar o pedido na etapa Overview.
Resultado esperado: sistema exibe mensagem de confirmação do pedido.

**TC-015 — Checkout com `problem_user` (regressão)**
Pré-condição: login realizado com `problem_user`.
Passos: adicionar produto ao carrinho → iniciar checkout → preencher dados → tentar avançar.
Resultado esperado: comportamento correto seria avançar para o resumo do pedido.
Resultado observado: divergência encontrada — ver [INC-001](./bugs.md#inc-001--checkout-não-avança-com-problem_user).

**TC-015a — Adicionar produto ao carrinho com `problem_user`**
Pré-condição: login realizado com `problem_user`.
Passos: acessar a vitrine → clicar em "Add to cart" em diferentes itens.
Resultado esperado: todos os itens são adicionados ao carrinho normalmente.
Resultado observado: divergência encontrada — ver [INC-002](./bugs.md#inc-002--add-to-cart-inoperante-em-itens-específicos-com-problem_user).

**TC-015b — Remover produto do carrinho pelo botão do item com `problem_user`**
Pré-condição: login realizado com `problem_user`; produto adicionado ao carrinho.
Passos: na vitrine, clicar no botão "Remove" do item já adicionado (sem entrar na página do carrinho).
Resultado esperado: o item é removido do carrinho diretamente pelo botão exibido no item.
Resultado observado: divergência encontrada — ver [INC-003](./bugs.md#inc-003--remoção-pelo-botão-do-item-não-funciona-fora-da-página-do-carrinho-com-problem_user).

**TC-016 — Iniciar checkout com carrinho vazio**
Pré-condição: login realizado com `standard_user`; carrinho sem itens.
Passos: acessar o carrinho vazio → clicar em Checkout → preencher informações → avançar até a finalização.
Resultado esperado: sistema deve impedir o início do checkout sem itens no carrinho.
Resultado observado: divergência encontrada — ver [INC-004](./bugs.md#inc-004--checkout-concluído-com-carrinho-vazio-standard_user).

## Conclusão

Dos 17 casos de teste executados, 13 confirmaram o comportamento esperado. Quatro revelaram divergências, registradas como incidentes em [`bugs.md`](./bugs.md):

- **INC-001** e **INC-002** — falhas de interação com `problem_user` (checkout e carrinho).
- **INC-003** — limitação de UX com `problem_user` (remoção fora da página do carrinho).
- **INC-004** — violação de regra de negócio com `standard_user` (checkout concluído com carrinho vazio), o achado de maior severidade por afetar o usuário-base da aplicação, não um usuário de teste especial.

Um caso (TC-013a) levantou um ponto de discussão de usabilidade — não um defeito — sobre para onde o cancelamento deveria direcionar o usuário na etapa final do checkout.

A cobertura atual se concentra em Autenticação e Checkout por serem as áreas de maior risco da aplicação (controle de acesso e conclusão de transação). Itens fora do escopo estão listados na seção correspondente e permanecem como candidatos a rodadas futuras de teste.

# Projeto — SauceDemo

Documentação de testes funcionais manuais para o [SauceDemo](https://www.saucedemo.com/), aplicação de e-commerce mantida pela Sauce Labs para prática e treinamento de QA. O projeto cobre os dois fluxos de maior risco da aplicação: autenticação e finalização de compra.

## Sumário
- [Escopo e Regras de Negócio](#escopo-e-regras-de-negócio)
- [Itens Fora do Escopo](#itens-fora-do-escopo)
- [Plano de Teste](#plano-de-teste)
- [Casos de Teste](#casos-de-teste)

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
- Campos obrigatórios ausentes devem impedir o avanço, com mensagem de erro.
- O resumo do pedido (Overview) deve refletir corretamente os itens e valores selecionados.
- O usuário pode cancelar o checkout a qualquer etapa e retornar ao carrinho/produtos.
- A finalização exibe mensagem de confirmação do pedido.
- **Comportamento conhecido:** o usuário `problem_user` não consegue concluir o checkout devido a uma falha no campo de sobrenome — bug documentado, não uma regra de negócio esperada.

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

**TC-013 — Cancelar o checkout**
Passos: iniciar o checkout → clicar em Cancel em qualquer etapa.
Resultado esperado: usuário retorna ao carrinho/produtos sem finalizar a compra.

**TC-014 — Finalizar compra com dados válidos**
Pré-condição: checkout preenchido corretamente.
Passos: confirmar o pedido na etapa Overview.
Resultado esperado: sistema exibe mensagem de confirmação do pedido.

**TC-015 — Checkout com `problem_user` (regressão)**
Pré-condição: login realizado com `problem_user`.
Passos: adicionar produto ao carrinho → iniciar checkout → preencher dados → tentar avançar.
Resultado esperado: comportamento correto seria avançar para o resumo do pedido.
Resultado observado: falha no campo de sobrenome impede a conclusão — bug documentado, não corrigido pela aplicação (comportamento intencional da Sauce Labs para prática de detecção de defeitos).

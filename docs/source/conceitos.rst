Conceitos
#########

Versionamento
=============

Introdução do conceito de versionamento as controllers, com isso o consumidor da API poderá escolher qual versão utilizar.
As versões suportadas pelo serviço sempre estarão alinhados com os releases do Korp ERP, por exemplo: 2023.2, 2023.1.
Qualquer alteração na API que contenha uma quebra obrigatoriamente gera uma versão nova. Se define como quebra o seguinte:

- Remover um endpoint (método publicado deixou de existir)
- Remover ou renomear um campo de entrada (propriedade em payload, propriedade na URL, etc)
- Remover ou renomear um campo de saída (propriedade no retorno)
- Adicionar um novo campo de entrada obrigatório
- Tornar obrigatório um campo de entrada anteriormente opcional
- Alterando o tipo de um campo de entrada ou saída
- Remover valores de enumerador (remover um valor de um enum)
- Adicionar uma nova regra de validação a um campo existente
- Alterar os requisitos de autenticação ou autorização (o endpoint precisa de uma permissão)
- Mudança significativa na regra de negócio (mudança impactante)

Política de versões
-------------------

As APIs deverão suportar pelo menos as últimas três versões.

Especificando uma versão
------------------------

O consumidor deverá apontar qual versão da API que quer utilizar enviando o header ``X-Korp-Api-Version``. Por exemplo:

.. code-block:: console

    curl --header "X-Korp-Api-Version:2023.1" https://gateway.korp.com.br/produtos

As requisições que não estiverem com o header ``X-Korp-Api-Version`` automaticamente utilizarão a versão do ambiente utilizado.

## core.ts

```typescript

const jsonFilePath = __dirname + '/data.temp.json'; // define o caminho completo do arquivo JSON
const list: string[] = await loadFromFile(); // cria uma lista de strings carregando os dados do arquivo

// função assíncrona responsável por carregar os dados do arquivo
async function loadFromFile() {
  try { // tenta executar a leitura do arquivo
    const file = Bun.file(jsonFilePath); // acessa o arquivo JSON
    const content = await file.text(); // lê o conteúdo do arquivo em texto
    return JSON.parse(content) as string[];  // converte o JSON em array de strings
  } catch (error: any) { // caso ocorra erro na leitura
    if (error.code === 'ENOENT') // verifica se o arquivo não existe
      return []; // retorna um array vazio
    throw error; // lança o erro para tratamento externo
  }
}

// função assíncrona responsável por salvar os dados
async function saveToFile() { 
  try { // tenta salvar os dados no arquivo
    await Bun.write(jsonFilePath, JSON.stringify(list)); // converte a lista em JSON e salva no arquivo
  } catch (error: any) { // caso ocorra erro
   throw new Error("Erro ao salvar os dados no arquivo: " + error.message); // retorna mensagem de erro
  }
}

// função responsável por adicionar itens na lista
async function addItem(item: string) {
  list.push(item); // adiciona o item ao array
  await saveToFile(); // salva as alterações no arquivo
}

// função responsável por listar os itens
async function getItems() {
  return list; // retorna o array completo
}

// função responsável por atualizar um item da lista
async function updateItem(index: number, newItem: string) { 
  if (index < 0 || index >= list.length) // verifica se o índice é válido
    throw new Error("Index fora dos limites"); // retorna erro caso o índice seja inválido

  list[index] = newItem; // substitui o valor no índice informado
  await saveToFile(); // salva as alterações
}

// função responsável por remover um item da lista
async function removeItem(index: number) {
  if (index < 0 || index >= list.length) // verifica se o índice é válido
    throw new Error("Index fora dos limites");  // retorna erro caso o índice seja inválido

  list.splice(index, 1); // remove o item do índice informado
  await saveToFile(); // salva as alterações
}

export default { addItem, getItems, updateItem, removeItem }; // exporta as funções para uso externo
````
---
## api.turma02.ts
```typescript

import todo from "./core.ts"; // importa as funções do arquivo core.ts

// inicializa o servidor
const server = Bun.serve({ 
  port: 3000, // define a porta utilizada pelo servidor

  // define as rotas da aplicação
  routes: { 
    "/": new Response(Bun.file("./public/index.html")), // rota principal que retorna o HTML

    // rota da API para operações da lista
    "/api/todo": { 

      // método GET para listar os itens
      GET: async () => { 
        const items = await todo.getItems(); // busca os itens salvos
        return Response.json(items); // retorna os itens em formato JSON
      },

      // método POST para adicionar um item
      POST: async (req) => { 
        const data = await req.json() as any; // converte os dados recebidos para JSON
        const item = data.item || null; // obtém o item enviado

        if (!item) // verifica se o item foi informado
          return Response.json('Por favor, forneça um item para adicionar.', { status: 400 }); // retorna erro caso não exista

        await todo.addItem(item); // adiciona o item na lista
        return Response.json(data); // retorna os dados adicionados
      },
    },

    // rota com parâmetro de índice para atualizar ou remover itens
    "/api/todo/:index": { 

      // método PUT para atualizar um item
      PUT: async (req) => { 
        const index = parseInt(req.params.index); // converte o índice da rota para número inteiro

        if (isNaN(index)) // verifica se o índice é um número válido
          return Response.json('Índice inválido. um número inteiro é esperado.', { status: 400 }); // retorna erro caso seja inválido

        const data = await req.json() as any; // converte os dados recebidos para JSON
        const newItem = data.newItem || null; // obtém o novo valor do item

        if (!newItem) // verifica se foi enviado um novo valor
          return Response.json('Por favor, forneça um novo item para atualizar.', { status: 400 }); // retorna erro caso não exista

        try { // tenta atualizar o item
          await todo.updateItem(index, newItem); // atualiza o item informado
          return Response.json(`Item no índice ${index} atualizado para "${newItem}".`); // retorna mensagem de sucesso
        } catch (error: any) { // caso ocorra erro
          return Response.json(error.message, { status: 400 }); // retorna a mensagem de erro
        }
      },

      // método DELETE para remover um item
      DELETE: async (req) => { 
        const index = parseInt(req.params.index); // converte o índice da rota para número inteiro

        if (isNaN(index)) // verifica se o índice é válido
          return Response.json('Índice inválido.', { status: 400 }); // retorna erro caso seja inválido 

        try { // tenta remover o item
          await todo.removeItem(index); // remove o item pelo índice informado
          return Response.json(`Item no índice ${index} removido com sucesso.`); // retorna mensagem de sucesso
        } catch (error: any) { // caso ocorra erro
          return Response.json(error.message, { status: 400 }); // retorna a mensagem de erro
        }
      },
    },
  },

  // executado quando nenhuma rota é encontrada
  async fetch(req) { 
    return new Response(`Not Found`, { status: 404 });  // retorna erro 404
  },
});

console.log(`Server running at http://localhost:${server.port}`); // exibe no console a porta em que o servidor está rodando
````

<template>
  <div class="kanban-board">
    <div
      v-for="(lista, status) in colunas"
      :key="status"
      class="column"
    >
      <h2>
        <span v-html="iconePorStatus[status]" class="icone"></span>
        {{ status }}
      </h2>

      <draggable
        v-model="colunas[status]"
        :group="'tarefas'"
        item-key="id"
        @end="salvarTarefas"
      >
        <template #item="{ element, index }">
          <div class="card">
            <template v-if="editandoId === element.id">
              <input
                v-model="editTexto"
                @keyup.enter="salvarEdicao(status, index)"
                @blur="salvarEdicao(status, index)"
                autofocus
              />
            </template>
            <template v-else>
              {{ element.text }}
              <div class="botoes">
                <button @click="editarTarefa(element)">✏️</button>
                <button @click="excluirTarefa(status, index)">🗑️</button>
              </div>
            </template>
          </div>
        </template>
      </draggable>
      
      <form @submit.prevent="adicionarTarefa(status)">
        <input v-model="novaTarefa[status]" placeholder="Nova tarefa..." />
        <button type="submit">+</button>
      </form>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable';

export default {
  components: { draggable },
  data() {
    return {
      novaTarefa: {
        'A Fazer': '',
        'Em Progresso': '',
        'Concluído': ''
      },
      colunas: {
        'A Fazer': [
          { id: 1, text: 'Criar layout' },
          { id: 2, text: 'Conectar API' }
        ],
        'Em Progresso': [
          { id: 3, text: 'Estilizar colunas' }
        ],
        'Concluído': [
          { id: 4, text: 'Configurar projeto' }
        ]
      },
      proximoId: 5,
      editandoId: null,
      editTexto: '',
      iconePorStatus: {
        'A Fazer': `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="#0052cc" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10" stroke="#0052cc" stroke-width="2" fill="none"/></svg>`,
        'Em Progresso': `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="#ffab00" viewBox="0 0 24 24"><path d="M12 2v20M2 12h20" stroke="#ffab00" stroke-width="2"/></svg>`,
        'Concluído': `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="#00b300" viewBox="0 0 24 24"><path d="M20 6L9 17l-5-5" stroke="#00b300" stroke-width="2" fill="none"/></svg>`
      }
    };
  },
  mounted() {
    const data = localStorage.getItem('tarefas');
    if (data) {
      this.colunas = JSON.parse(data);
      const allIds = Object.values(this.colunas)
        .flat()
        .map(t => t.id);
      this.proximoId = allIds.length ? Math.max(...allIds) + 1 : 1;
    }
  },
  methods: {
    adicionarTarefa(status) {
      const texto = this.novaTarefa[status].trim();
      if (texto) {
        this.colunas[status].push({
          id: this.proximoId++,
          text: texto
        });
        this.novaTarefa[status] = '';
        this.salvarTarefas();
      }
    },
    editarTarefa(tarefa) {
      this.editandoId = tarefa.id;
      this.editTexto = tarefa.text;
    },
    salvarEdicao(status, index) {
      if (this.editTexto.trim()) {
        this.colunas[status][index].text = this.editTexto.trim();
        this.salvarTarefas();
      }
      this.editandoId = null;
      this.editTexto = '';
    },
    excluirTarefa(status, index) {
      this.colunas[status].splice(index, 1);
      this.salvarTarefas();
    },
    salvarTarefas() {
      localStorage.setItem('tarefas', JSON.stringify(this.colunas));
    }
  }
};
</script>

<style scoped>
.kanban-board {
  display: flex;
  justify-content: space-between;
  padding: 2rem;
  gap: 1rem;
  background-color: #f4f5f7;
  min-height: 100vh;
  font-family: 'Segoe UI', sans-serif;
}

.column {
  background: #ebecf0;
  padding: 1rem;
  border-radius: 8px;
  flex: 1;
  min-width: 250px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.column h2 {
  margin-top: 0;
  color: #172b4d;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.icone {
  display: inline-flex;
  vertical-align: middle;
}

.card {
  background: white;
  color:#172b4d;
  margin-bottom: 0.5rem;
  padding: 1rem;
  border-radius: 6px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.2);
  cursor: grab;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card input {
  flex: 1;
  padding: 0.3rem 0.5rem;
  font-size: 1rem;
}

.botoes button {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.1rem;
  margin-left: 0.5rem;
  transition: color 0.2s;
}

.botoes button:hover {
  color: #0052cc;
}

form {
  display: flex;
  margin-top: 1rem;
}

input {
  flex: 1;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px 0 0 4px;
  background-color: rgb(229, 236, 236);
}

button {
  background: #0052cc;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}
</style>

<template>
    <div>
      <h2>Listagem de Knights</h2>
      <label>
        <input type="checkbox" v-model="filtroHerois" @change="filtrarKnights">
        Filtrar por heróis
      </label>
      <table>
        <thead>
            <tr>
            <th>Nome</th>
            <th>Nickname</th>
            <th>Idade</th>
            <th>Armas</th>
            <th>Atributo</th>
            <th>Ataque</th>
            <th>Exp</th>
            <th>Editar</th>
            <th>Excluir</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="knight in knightsFiltered" :key="knight.id">
            <td>{{ knight.nome }}</td>
            <td>
                <span v-if="knight.editando">
                    <input type="text" v-model="knightEdit.nickname" @blur="atualizarKnight(knight)">
                </span>
                <span v-else>
                    {{ knight.nickname }}
                </span>
            </td>
            <td>{{ knight.idade }}</td>
            <td>{{ knight.armas }}</td>
            <td>{{ knight.atributo }}</td>
            <td>{{ knight.ataque }}</td>
            <td>{{ knight.exp }}</td>
            <td>
                <span v-if="!knight.editando">
                    <button @click="editarKnight(knight)">Editar Nickname</button>
                </span>
                <span v-else>
                    <button @click="atualizarKnight(knight)">Salvar</button>
                </span>
            </td>
            <td>
                <span v-if="!knight.hero">
                    <button @click="deletarKnight(knight.id)">Tornar Heroi</button>
                </span>
                <span v-else>
                    Heroí
                </span>
            </td>
            </tr>
        </tbody>
      </table>
      <hr>
      <h2>Criar Novo Herói</h2>
      <form @submit.prevent="criarHeroi" class="hero-form">
        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="name">Nome:</label>
            <input type="text" id="name" v-model="novoHeroi.name" required>
        </div>
        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="nickname">Nickname:</label>
            <input type="text" id="nickname" v-model="novoHeroi.nickname" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="birthday">Data de Nascimento:</label>
            <input type="date" id="birthday" v-model="novoHeroi.birthday" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="strength">Força:</label>
            <input type="number" id="strength" v-model="novoHeroi.attributes.strength" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="dexterity">Destreza:</label>
            <input type="number" id="dexterity" v-model="novoHeroi.attributes.dexterity" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="constitution">Constituição:</label>
            <input type="number" id="constitution" v-model="novoHeroi.attributes.constitution" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="intelligence">Inteligência:</label>
            <input type="number" id="intelligence" v-model="novoHeroi.attributes.intelligence" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="wisdom">Sabedoria:</label>
            <input type="number" id="wisdom" v-model="novoHeroi.attributes.wisdom" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="charisma">Carisma:</label>
            <input type="number" id="charisma" v-model="novoHeroi.attributes.charisma" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="keyAttribute">Atributo Chave:</label>
            <select id="keyAttribute" v-model="novoHeroi.keyAttribute" required>
                <option value="strength">Força</option>
                <option value="dexterity">Destreza</option>
                <option value="constitution">Constituição</option>
                <option value="intelligence">Inteligência</option>
                <option value="wisdom">Sabedoria</option>
                <option value="charisma">Carisma</option>
            </select>
        </div>

        <hr>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="weaponName">Nome da Arma:</label>
            <input type="text" id="weaponName" v-model="novoHeroi.weapons[0].name" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="weaponMod">Modificador da Arma:</label>
            <input type="number" id="weaponMod" v-model="novoHeroi.weapons[0].mod" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="weaponAttr">Atributo da Arma:</label>
            <input type="text" id="weaponAttr" v-model="novoHeroi.weapons[0].attr" required>
        </div>

        <div style="display: flex; justify-content: start; align-items: center;">
            <label for="weaponEquipped">Equipada:</label>
            <input type="checkbox" id="weaponEquipped" v-model="novoHeroi.weapons[0].equipped">
        </div>
        
        <button type="submit">Criar Herói</button>
      </form>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        filtroHerois: false,
        editando: false,
        knights: [],
        knightEdit: {
            id: null,
            nickname: '',
        },
        novoHeroi: {
            name: '',
            nickname: '',
            birthday: '',
            weapons: [
                {
                    name: '',
                    mod: 0,
                    attr: '',
                    equipped: false,
                },
            ],
            attributes: {
            strength: 0,
            dexterity: 0,
            constitution: 0,
            intelligence: 0,
            wisdom: 0,
            charisma: 0,
            },
            keyAttribute: '',
        },
      };
    },
    computed: {
        knightsFiltered() {
            return this.knights;
        },
    },
    methods: {
      async filtrarKnights() {
        this.fetchKnights();
      },
      async editarKnight(knight) {
        this.knightEdit.id = knight.id;
        this.knightEdit.nickname = knight.nickname;
        knight.editando = true;
      },
      async atualizarKnight() {
        try {
            await axios.patch(`http://localhost:3000/knights/${this.knightEdit.id}`, {
            nickname: this.knightEdit.nickname,
            });
            // Recarregar a lista de knights após a atualização
            this.fetchKnights();
        } catch (error) {
            console.error('Erro ao atualizar knight:', error);
        }
      },
      async deletarKnight(id) {
        try {
          await axios.delete(`http://localhost:3000/knights/${id}`);
          // Atualizar lista de knights após exclusão
          this.fetchKnights();
        } catch (error) {
          console.error('Erro ao deletar knight:', error);
        }
      },
      async fetchKnights() {
        try {
          const url = this.filtroHerois ? 'http://localhost:3000/knights?filter=heroes' : 'http://localhost:3000/knights';
          const response = await axios.get(url);
          this.knights = response.data;
        } catch (error) {
          console.error('Erro ao buscar knights:', error);
        }
      },
      async criarHeroi() {
        try {
            await axios.post('http://localhost:3000/knights', this.novoHeroi);
            // Limpar o formulário após a criação do herói
            this.novoHeroi = {
            name: '',
            nickname: '',
            birthday: '',
            weapons: [
                {
                    name: '',
                    mod: 0,
                    attr: '',
                    equipped: false,
                },
            ],
            attributes: {
                strength: 0,
                dexterity: 0,
                constitution: 0,
                intelligence: 0,
                wisdom: 0,
                charisma: 0,
            },
            keyAttribute: '',
            };
            // Recarregar a lista de knights após a criação do herói
            this.fetchKnights();
        } catch (error) {
            console.error('Erro ao criar herói:', error);
        }
        },
    },
    mounted() {
      this.fetchKnights();
    },
  };
  </script>
  
  <style scoped>
    table {
        width: 100%;
        border-collapse: collapse;
        margin-top: 30px;
    }
    
    th, td {
        border: 1px solid #dddddd;
        text-align: left;
        padding: 8px;
    }
    
    th {
        background-color: #f2f2f2;
    }
    
    tr:nth-child(even) {
        background-color: #f2f2f2;
    }
    
    tr:hover {
        background-color: #dddddd;
    }

    .hero-form {
        max-width: 400px;
        margin: 0 auto;
        margin-top: 20px;
    }

        .hero-form label {
        display: block;
        margin-bottom: 5px;
        min-width: 150px;
        }

        .hero-form input[type="text"],
        .hero-form input[type="date"],
        .hero-form input[type="number"] {
        width: 100%;
        padding: 8px;
        margin-bottom: 15px;
        border: 1px solid #ccc;
        border-radius: 4px;
        box-sizing: border-box;
        }

        .hero-form button[type="submit"] {
        background-color: #4CAF50;
        color: white;
        padding: 10px 20px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        }

        .hero-form button[type="submit"]:hover {
        background-color: #45a049;
        }
  </style>
  
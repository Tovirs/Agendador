<template>
  <div id="agendar">
    <nav class="nav-wrapper red darken-4">
      <span class="brand-logo center">Agendamento de Provas</span>
    </nav>

    <div class="container">
      <div>
        <p id="textoAjuda" class="center-align">Clique numa disciplina abaixo para começar a agendar</p>
      </div>

      <ul v-for="disciplina in disciplinas" class="collapsible popout" data-collapsible="acordion" :id="disciplina.codigo">
        <li>
          <div @click="abrirCollapsible(disciplina.descricao,disciplina.codigo)" class="collapsible-header">
            {{ disciplina.descricao }}
            <!-- <i v-bind:style="{opacity: checkOpacity}" class="material-icons right">check</i> -->
          </div>
          <div class="collapsible-body">
              <h5 :id="'resumoHeader-' + disciplina.codigo" align="center"><strong>Prova Agendada</strong></h5>
            <table :id="'resumo-' + disciplina.codigo" class="highlight centered">
              <thead>
                <tr>
                  <th>Unidade</th>
                  <th>Data</th>
                  <th>Sala</th>
                  <th></th>
                </tr>
              </thead>
            </table>
            <select :id="'select-' + disciplina.codigo" class="browser-default" v-model="unidadeSelecionada" v-on:change="exibicaoDaListaDeHorarios(disciplina.codigo)"
              required>
              <option value="" disabled selected>Unidade:</option>
              <option v-for="unidade in disciplina.unidades" v-bind:value="unidade">
                {{ unidade.descrição }}
              </option>
            </select>
            <div :id="'horario-' + disciplina.codigo">
              <table class="highlight centered">
                <thead>
                  <tr>
                    <th>Data</th>
                    <th>Horario</th>
                    <th>Sala</th>
                  </tr>
                </thead>
                <tbody>
                  <tr @click="abrirModalDeConfirmacaoDeDisciplina(disciplina, horario.data, horario.sala)" class="collection-item center modal-trigger blocoHorario"
                    v-for="horario in unidadeSelecionada.horarios">
                    <td>{{ horario.data | dataFormatada }}</td>
                    <td> {{ horario.data | horarioFormatado }}</td>
                    <td> Sala {{ horario.sala }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </li>
      </ul>

      <div id="modal" class="modal">
        <div class="modal-content">
          <h4 align="center">Confirmar Disciplina?</h4>
          <p align="center">
            Disciplina: {{ selecaoExibidaNoModal.disciplinaSelecionada }}<br> Unidade: {{ selecaoExibidaNoModal.unidadeSelecionada
            }}
            <br> Data: {{ selecaoExibidaNoModal.dataSelecionada }}<br> Sala: {{ selecaoExibidaNoModal.salaSelecionada }}

            <!-- Não sei porque dá erro formatando dessa forma!! ↓↓↓↓↓↓↓ -->
            <!-- Data: {{ selecaoExibidaNoModal.dataSelecionada | dataFormatada }}<br> -->
            <!-- Horario: {{ selecaoExibidaNoModal.dataSelecionada | horarioFormatado }}<br> -->
          </p>
        </div>

        <div class="modal-footer">
          <a @click="salvarDadosNoLocalStorage()" href="#!" class="modal-action modal-close waves-effect waves-green btn-flat">Confirmar</a>
          <a href="#!" class="modal-action modal-close waves-effect waves-green btn-flat">Cancelar</a>
        </div>
      </div>
    </div>
  </div>

</template>



<script>
import disciplinasJSON from '../../dados_json/disciplinascompleto.json'
import moment from 'moment'


export default {
    name: 'Agendar',
    data () {
      return {
        disciplinas: [],
        unidadeSelecionada: [],
        selecaoExibidaNoModal: {
          disciplinaSelecionada: '',
          unidadeSelecionada: '',
          dataSelecionada: '',
          salaSelecionada: ''
        },
        disciplinaSelecionada: {},
        objResumoAgendamento: [],
        objResumoAgendamentoCollapsible: [],
        descricaoTemp: '',
        codigoTemp: '',
      }
    },
    methods: {
      abrirCollapsible: function (descricao, codigo) {
        this.descricaoTemp = descricao; //armazena a descricao da disciplina clicada temporariamente
        this.codigoTemp = codigo; //armazena o codigo da disciplina clicada temporariamente
        $('.collapsible').collapsible('close', 0); // Devido ao fato de o horário de uma disciplina aparecer nos collapsibles de outras,
                                                   // este código fecha os outros collapsibles.
                                                   // Precisamos rever isso, para não precisarmos fechar.
                                                   // Além disso, dessa forma, depois de aberto, o usuário só consegue fechar o collapsible
                                                  // quando abre outro.
        $('#resumoHeader-' + codigo).hide(); //Escondendo o paragráfo do resumo
        $('#resumo-' + codigo).hide(); //Escondendo o resumo
        $('#horario-' + codigo).hide(); //Escondendo os horários da disciplina
        this.mostrarResumoDoAgendamento(); //chama a função de mostrar o resumo ao abrir o collapsible
        $('#' + codigo).collapsible('open'); //abrindo o collapsible clicado
      },
      abrirModalDeConfirmacaoDeDisciplina: function (disciplina, data, sala) {
        this.selecaoExibidaNoModal.disciplinaSelecionada = disciplina.descricao;
        this.selecaoExibidaNoModal.unidadeSelecionada = this.unidadeSelecionada.descrição;
        this.selecaoExibidaNoModal.dataSelecionada = data;
        this.selecaoExibidaNoModal.salaSelecionada = sala;
        $('#modal').modal('open');
      },
      exibicaoDaListaDeHorarios: function (id) {
        $('#horario-'+id).show();
      },
      salvarDadosNoLocalStorage: function () {
        //↓ Cria o objeto que será armazenado no local storage
        var objDisciplinaAgendada = {
          disciplinaAgendada: this.selecaoExibidaNoModal.disciplinaSelecionada,
          unidadeAgendada: this.selecaoExibidaNoModal.unidadeSelecionada,
          dataAgendada: this.selecaoExibidaNoModal.dataSelecionada,
          salaAgendada: this.selecaoExibidaNoModal.salaSelecionada
        }
        //Verifica se já há disciplinas agendadas
        if(localStorage.getItem("Agendamentos") === null) {
          //Caso não haja matérias agendadas, insere o objeto em um array e o armazena no local storage
          var arrayDeAgendamentos = [objDisciplinaAgendada];
          localStorage.setItem("Agendamentos",JSON.stringify(arrayDeAgendamentos));
          this.mostrarResumoDoAgendamento();
        }
        else
        {
          //Caso haja matérias agendadas, insere o objeto no array já criado
          var arrayDeAgendamentos = JSON.parse(localStorage.getItem("Agendamentos"));
          arrayDeAgendamentos.push(objDisciplinaAgendada);
          localStorage.setItem("Agendamentos",JSON.stringify(arrayDeAgendamentos));
          this.mostrarResumoDoAgendamento();
        }
      },
      mostrarResumoDoAgendamento: function () {
        //Pega os agendamentos do banco de dados
        this.objResumoAgendamento = JSON.parse(localStorage.getItem("Agendamentos"));
        
        if (this.objResumoAgendamento){ //teste para ver se pegou algo
          var qtdDisciplinas = Object.keys(this.objResumoAgendamento).length; //armazenando o tamanho do objeto para ser usado no for
          for (var i = 0; i < qtdDisciplinas; i++) {
            if (this.descricaoTemp == this.objResumoAgendamento[i].disciplinaAgendada) { //testa se a descricao da disciplina clicada no collapsible é igual ao que está guardado no local storage
              $('#resumoHeader-' + this.codigoTemp).show(); //deixando o header do resumo visivel
              $('#resumo-' + this.codigoTemp).show(); //deixando a tabela com o resumo vísivel
              //gambiarra para inserir o resumo do agendamento na tabela
              $('#resumo-' + this.codigoTemp).append("<tbody><tr><td>" + this.objResumoAgendamento[i].unidadeAgendada + "</td> <td>" + this.objResumoAgendamento[i].dataAgendada + "</td> <td>" +this.objResumoAgendamento[i].salaAgendada + "</td> <td><a onclick=alert('botão_comunista...não_funciona'); style='font-size:14px;' class='red darken-4 waves-effect waves-light btn'>Cancelar Agendamento</a></td></tr></tbody>"); 
              $('#select-' + this.codigoTemp).hide(); //esconde o select
              $('#horario-' + this.codigoTemp).hide(); //esconde os horários
              $('#resumo-' + this.codigoTemp).prop('id', 'XGH'); //gambiarra gambiarrosa gambiarrenta para mudar o ID e não inserir o agendamento novamente ao clicar duas vezes no collapsible
            }
          }
        }
      }



      // A parte do check deve ser refeita e inserida na função salvarDadosNoLocalStorage
      // salvar: function () {
      //   if (localStorage.getItem("agendado")) {
      //     this.DiscipUnidHora = JSON.parse(localStorage.getItem("agendado"));
      //   }

      //   this.DiscipUnidHora.push(this.discipSelec, this.unidSelec, this.horaData, this.horaSala);
      //   localStorage.setItem("agendado", JSON.stringify(this.DiscipUnidHora));
      //   this.horarioModal.splice(0); 
      //   localStorage.setItem("check"+this.checkSelec, "true");         
      //   if (this.checkSelec == 0) {
      //     this.checkOpacity0 = 100;
      //   }
      //   if (this.checkSelec == 1) {
      //     this.checkOpacity1 = 100;
      //   }
      //   if (this.checkSelec == 2) {
      //     this.checkOpacity2 = 100;
      //   }
      // }
    },
    mounted: function() {
      $('.collapsible').collapsible();
      $('select').material_select();
      $('.modal').modal();
      
      this.disciplinas = disciplinasJSON.disciplinas;
      // if (localStorage.getItem("check0")) {
      //   this.checkOpacity0 = 100;
      //   }
      // if (localStorage.getItem("check1")) {
      //   this.checkOpacity1 = 100;
      // }
      // if (localStorage.getItem("check2")) {
      //   this.checkOpacity2 = 100;
      // }
    },
    filters: {
      dataFormatada: function(data) {
        //Precisamos melhorar essa parte. Consegui converter as datas somente
        //a partir de milisegundos. Por isso o '+000'
        var dataParse = JSON.parse(data+'000')
        return moment(dataParse).format('DD/MM/YYYY')
      },
      horarioFormatado: function(data) {
        var dataParse = JSON.parse(data+'000')
        return moment(dataParse).format('hh:mm')
      }
    }
}
</script>

<style>
.blocoHor{
cursor: pointer;
}
#check{
  display: none;
  float: right;
  color:#00ff00;
  
}
#check2{
  display: none;
  float: right;
  color:#00ff00;
}
#check3{
  display: none;
  float: right;
  color:#00ff00;
}
  .collapsible-header {
    position: relative;
  }

  a.collection-item {
    color: black !important;
  }

  #textoAjuda {
    font-size: 25px;
  }
</style>
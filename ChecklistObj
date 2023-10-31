/** @OnlyCurrentDoc */

/** 
 * NOTAS SOBRE A VERSÃO
 * 
 * O dado userName busca o usuário ativo para registrá-lo através da função set.
 * Ainda não há um get para o dado na tabela. Não sei se deve haver. O userName e
 * seu registro foram desativados pois não há outros usuários ainda. Desativados em: declaração
 * de variaveis globais e em set_inspecao
 * 
 * 
 * Nesta versão ela está funcional para múltiplos usuários.
 * Basta copiar a URL acrescentando /copy depois da id do documento e compartilhar por email.
 * Ao abrir, o usuário deverá criar uma copia (basta clicar no email e esta opção será sugerida).
*/

/**Variáveis universais */

var opcoes = { year: 'numeric', month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit' };
var data = new Date().toLocaleDateString('pt-Br',opcoes);
// var dataCompleta = new Date().toLocaleString('pt-BR',opcoes)
//URL da planilha-registro. Onde são salvos registros de inspeções de todas instâncias de checklist
var URL = 'https://docs.google.com/spreadsheets/d/1Xk2tyhYnu89-S6kngz0L1UDIkd1wVLje9NLaS23iQOo/edit';
const userName = Session.getActiveUser().getEmail();
const masterUserName = '11.guerra.pedro@gmail.com' //Username usada para testes de maior permissividade.
var temp_proj_num
var cache = CacheService.getScriptCache();/** Não tem variáveis globais no GAS do jeito convencional. A maneira de alterar e resgatar valores dinamicamente é através do cache */

function editarNumProj(){
  var ui = SpreadsheetApp.getUi()
  var response = ui.prompt('Editar número do projeto: ').getResponseText()
  var s  = SpreadsheetApp.getActive()
  setProperty('num_proj',response)
  s.getRange('Checklist!E2').setValue(response)

}

function obterDataHoraFormatada() {
  var opcoes = { year: 'numeric', month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit' };
  var dataHoraFormatada = dataCompleta.toLocaleString('pt-BR', opcoes);
  return dataHoraFormatada;
}

function minhaFuncao() {
  // var dataHora = obterDataHoraFormatada();
  Logger.log("Data e hora formatadas: " + dataCompleta);
  Logger.log("Data SIMPLES: " + dataCompleta.slice(0,10));
}



function aoEditar() {
  /*Essa função é um gatilho - Observador */
  var spreadsheet = SpreadsheetApp.getActive()
  Logger.log('teste: ' + typeof cache.get('numProjeto') + 'teste 2: ' + typeof spreadsheet.getRange('Checklist!E2').getValue())
  Logger.log(spreadsheet.getRange('Checklist!E2').getValue() == cache.get('numProjeto'))
  resetOnChange()
  liste_pendencias();
};


function resetOnChange(){
  /**
   * Reseta o ID e a Data na planilha checklist no caso de alteração do número de projeto;
   * ou
   * reseta o ID em caso de alteração da Data na planilha checklist. (Um projeto pode ter mais de uma inspeção e em datas diferentes)
   * 
   * Isso evita que seja feita um get_InspObj() ou Substituir() com ID ou data incorretos.
  */
  var spreadsheet = SpreadsheetApp.getActive()
   if(!getProperty('resetado') && (spreadsheet.getRange('Checklist!E2').getValue() != getProperty('num_proj'))){
    //Apaga a data e o ID na planilha checklist se você alterar o número do projeto.
    spreadsheet.getRange('Checklist!F2').setValue(null);
    spreadsheet.getRange('Checklist!C1').setValue(null);
  }
  if(!getProperty('resetado') && (spreadsheet.getRange('Checklist!F2').getValue().toLocaleString('pt-Br',opcoes) != getProperty('data_insp'))){
    //Apaga o ID se você alterar a data na planilha checklist
    spreadsheet.getRange('Checklist!C1').setValue(null);
  }
}

function tabelar() {

  // var linhaObjeto = 
  // var objeto = JSON.parse(tabRegistro.getRange(`C${}`).getValue())
  var linha = get_linha_vazia('controle')
  var tabRegistro = SpreadsheetApp.getActive().getSheetByName('registros')
  var s = SpreadsheetApp.getActive().getSheetByName('controle')
  var range = s.getRange(`E${linha}:AC${linha}`)
  s.getRange(`A${linha}`).setValue(objeto.projeto)
  s.getRange(`B${linha}`).setFormula(`=SPLIT(A${linha};".")`)
  var valores = []
  
  for (var i = 0; i < objeto.checklist.length; i++) {
    var pendencias = objeto.checklist[i];
    if (pendencias[0] == true) {
      valores.push('NA');
    } else if (pendencias[1] == true) {
      valores.push('OK');
    } else if (pendencias[1] == false && pendencias[2] == false) {
      valores.push('Não conforme');
    } else {
      valores.push('Falta');
    }
  }

  range.setValues([valores])

}


function setProperty(item, valor) {
  // Define uma propriedade de script
  /**setProperty e getProperty são funções para salvar e recuperar os valores desejados de cada instância
   * de execução da planilha Checklist. Cada usuário salva, nesse caso, a data de inspeção e número do projeto
   * da inspeção atual.
   */
  
  var userProperties = PropertiesService.getUserProperties();

  userProperties.setProperty(item, valor);
}

function getProperty(item) {
  // Recupera uma propriedade de script
  var userProperties = PropertiesService.getUserProperties();
  var valor = userProperties.getProperty(item);
  if (item == 'resetado' && valor === 'true'){
    /** Gambiarra porque esse método da Classe userProperties só opera com Strings ou null */
    return true
  }else if(item == 'resetado'){
    return false
  }else{
    return valor
  }
}


function alertaProjEnc() {
  var planilhaRegistro = SpreadsheetApp.openByUrl(URL)
  var checkEncerramento = SpreadsheetApp.getActive().getRange('Checklist!B5:D5').getValues().toString();
  var linhaVazia = false;
  // var destinatario = "11.guerra.pedro@gmail.com";
  var destinatario = "pedro.guerra@hect.com.br; priscila.ferreira@hect.com.br"
  var assunto = "Projetos terminados";
  var corpoDoEmail = "Estes projetos já extrapolaram a data de término acordado: \n\n"
  var projsEncerrados = new Array()
  var prazo_ext;

  for(var i = 2; !linhaVazia; i++){
    prazo_ext = new Date( planilhaRegistro.getRange('Programa!B' + i).getValue())
      Logger.log('Ta na linha: ' + i)
    if(planilhaRegistro.getRange(`Programa!A${i}:B${i}`).isBlank() && checkEncerramento != [false,true,false]){
      linhaVazia = true;
    }else if (prazo_ext < new Date()) {
        projsEncerrados.push(planilhaRegistro.getRange(`Programa!A${i}`).getValue())
        planilhaRegistro.getRange('Programa!C' + i).setValue('Enviado')

      }
  }
  corpoDoEmail += projsEncerrados.toString()
  GmailApp.sendEmail(destinatario, assunto, corpoDoEmail);
  Logger.log(projsEncerrados)  
}


function abrirProj(){
  var spreadsheet = SpreadsheetApp.getActive()
  try{
     var projeto = spreadsheet.getRange('Checklist!E2').getValue().toString().split('.')
   var num = projeto[0].slice(0,3);
   var ciclo;
    if(projeto[1] <= 9){
     ciclo = projeto[1][1]
    }else{
    ciclo = projeto[1]
    }
   var link = `https://hectcombr.sharepoint.com/sites/p${num}/C${ciclo}`
   Logger.log(link)

  var htmlOutput = HtmlService.createHtmlOutput(`<script>window.open("${link}", "_blank"); google.script.host.close();</script>`);

  SpreadsheetApp.getUi().showModalDialog(htmlOutput, "ABRINDO PASTA DO PROJETO...");
  }catch(error){
    if(error.message == "Cannot read properties of undefined (reading '1')"){
      // Logger.log('Erro: Projeto não informado')
      SpreadsheetApp.getUi().alert("Erro: Projeto não informado");
    }
  }
}

function zerar(param_planilha, zerarProjDataId = true) {
  var planilha = (param_planilha === undefined) ? SpreadsheetApp.getActiveSheet().getSheetName() + '!' : param_planilha;
  var spreadsheet = SpreadsheetApp.getActiveSheet()
  spreadsheet.getRange(planilha + 'B4:D28').setValue('FALSE')
  spreadsheet.getRange(planilha + 'B13:B14').setFormula('=B12')
  spreadsheet.getRange(planilha + 'F4:F30').setValue(null)
  spreadsheet.getRange(planilha + 'G2').setValue(null)
  if (zerarProjDataId) {
    spreadsheet.getRange(planilha + 'C1').setValue(null);
    spreadsheet.getRange(planilha + 'E2').setValue(null);
    spreadsheet.getRange(planilha + 'F2').setValue(null);
  }
  setProperty('resetado',true)
  liste_pendencias();

}

function liste_pendencias() {
  var spreadsheet = SpreadsheetApp.getActive();
  // var qntpend = 0;
  var j = 1;
  var relatorio = '';
  var relatorioToCopy = '';
  var doc_ausente = '';
  var observacao = '';
  for (var i = 4; i <= 13; i++) { //P3.01
    if ((!spreadsheet.getRange('C' + i).getValue() || spreadsheet.getRange('D' + i).getValue()) && (!spreadsheet.getRange('B' + i).getValue())) {
      // qntpend++; 
      observacao = spreadsheet.getRange('F' + i).getValue() == '' ? '' : (': ' + spreadsheet.getRange('F' + i).getValue())
      doc_ausente = (spreadsheet.getRange('D' + i).getValue() ? 'Falta ' : 'Pendência com ');
      // Linha abaixo é de teste
      // if (j == 1) relatorio += data;
      relatorio += j + '. ' + doc_ausente + spreadsheet.getRange('E' + i).getValue() + observacao + "\n";
      j++;
    };

  };
  if (relatorio != '') relatorioToCopy = `Em 'Padrões'\n` + relatorio;
  // spreadsheet.getRange('Checklist!D20').setValue(relatorio);

  relatorio = '';
  for (var i = 14; i < 16; i++) { //Entregas
    if ((!spreadsheet.getRange('C' + i).getValue() || spreadsheet.getRange('D' + i).getValue()) && (!spreadsheet.getRange('B' + i).getValue())) {
      // qntpend++;
      observacao = spreadsheet.getRange('F' + i).getValue() == '' ? '' : (': ' + spreadsheet.getRange('F' + i).getValue())
      doc_ausente = (spreadsheet.getRange('D' + i).getValue() ? 'Falta ' : 'Pendência com ');
      // if (j == 1) relatorio += data;
      relatorio += j + '. ' + doc_ausente + spreadsheet.getRange('E' + i).getValue() + observacao + "\n";
      j++;
    };

  };
  if (relatorio != '') relatorioToCopy += `\nEm 'Entregas':\n` + relatorio;
  // spreadsheet.getRange('Checklist!D22').setValue(relatorio);

  relatorio = '';
  for (var i = 16; i <= 19; i++) { //Jur
    if ((!spreadsheet.getRange('C' + i).getValue() || spreadsheet.getRange('D' + i).getValue()) && (!spreadsheet.getRange('B' + i).getValue())) {
      doc_ausente = (spreadsheet.getRange('D' + i).getValue() ? 'Falta ' : 'Pendência com ');
      observacao = spreadsheet.getRange('F' + i).getValue() == '' ? '' : (': ' + spreadsheet.getRange('F' + i).getValue())
      // if (j == 1) relatorio += data;
      relatorio += j + '. ' + doc_ausente + spreadsheet.getRange('E' + i).getValue() + observacao + "\n";
      j++;
    };

  };
  if (relatorio != '') relatorioToCopy += `\nPendências de 'jur'\n` + relatorio;
  // spreadsheet.getRange('Checklist!H20').setValue(relatorio);

  relatorio = '';
  for (var i = 20; i <= 24; i++) { //Risco
    if ((!spreadsheet.getRange('C' + i).getValue() || spreadsheet.getRange('D' + i).getValue()) && (!spreadsheet.getRange('B' + i).getValue())) {
      observacao = spreadsheet.getRange('F' + i).getValue() == '' ? '' : (': ' + spreadsheet.getRange('F' + i).getValue())
      doc_ausente = (spreadsheet.getRange('D' + i).getValue() ? 'Falta ' : 'Pendência com ');
      // if (j == 1) relatorio += data;
      relatorio += j + '. ' + doc_ausente + spreadsheet.getRange('E' + i).getValue() + observacao + "\n";
      j++;
    };

  };
  // if (relatorio != '') relatorioToCopy += `\nPendências de 'Riscos'\n` + relatorio;
  if (relatorio != '') relatorioToCopy += `\n\n` + relatorio;
  // spreadsheet.getRange('Checklist!L20').setValue(relatorio);

  relatorio = '';
  for (var i = 25; i <= 28; i++) { //Proj_dados
    if ((!spreadsheet.getRange('C' + i).getValue() || spreadsheet.getRange('D' + i).getValue()) && (!spreadsheet.getRange('B' + i).getValue())) {
      observacao = spreadsheet.getRange('F' + i).getValue() == '' ? '' : (': ' + spreadsheet.getRange('F' + i).getValue())
      doc_ausente = (spreadsheet.getRange('D' + i).getValue() ? 'Falta ' : 'Pendência com ');
      // if (j == 1) relatorio += data;
      relatorio += j + '. ' + doc_ausente + spreadsheet.getRange('E' + i).getValue() + observacao + "\n";
      j++;
    };

  };
  if (relatorio != '') relatorioToCopy += `\nEm 'Recebidos e/ou Comunicação com cliente'\n` + relatorio;

  relatorio = '';
  if (spreadsheet.getRange('Checklist!F29').getValue() != '' || !(spreadsheet.getRange('Checklist!F29').isBlank())) {
    relatorio += spreadsheet.getRange('Checklist!F29').getValue();
  };
  if (relatorio != '') relatorioToCopy += `\nObservações: \n` + relatorio;
  relatorio = '';

  if (relatorioToCopy == '') {
    spreadsheet.getRange('Checklist!G4').setValue('Não há pendências')
  } else {
    spreadsheet.getRange('Checklist!G4').setValue(relatorioToCopy.trim())
  }
};

function na_jur() {
  var spreadsheet = SpreadsheetApp.getActive();
  if (spreadsheet.getRange('B16:B19').getValue() == false) {
    spreadsheet.getRange('B16:B19').setValue(true);
  } else if (spreadsheet.getRange('B16:B19').getValue() == true) {
    spreadsheet.getRange('B16:B19').setValue(false);
  }
  liste_pendencias()
};

function na_risco() {
  var spreadsheet = SpreadsheetApp.getActive();
  if (spreadsheet.getRange('B20:B24').getValue() == false) {
    spreadsheet.getRange('B20:B24').setValue(true);
  } else if ((spreadsheet.getRange('B20:B24').getValue() == true)) {
    spreadsheet.getRange('B20:B24').setValue(false);
  }
  liste_pendencias()
}


function set_inspObj(linhaRegistro = null) {

  if (actionCheck("Deseja salvar essa Inspeção?") == "YES") {
    var planilhaExterna = SpreadsheetApp.openByUrl(URL)
    var planilhaChecklist = SpreadsheetApp.getActive()
    const nome_Checklist = 'Checklist!'
    const nome_planExterna = 'registros_checklist!'
    /**Captura de valores para o objeto 'inspecao' 
     * 
     * Se a planilha Checklist tiver sua tabela alterada, um ajuste correspondente deve ser feito
     * no range abaixo. 
     * 
     * ### Atenção para lista_checklist e lista_pendencias
    */
    var proj = planilhaChecklist.getRange(nome_Checklist + 'E2').getValue()
    var observacao_projeto = planilhaChecklist.getRange(nome_Checklist + 'F29').getValue()
    var observacao_qualidade = planilhaChecklist.getRange(nome_Checklist + 'F30').getValue()
    var lista_checklist = planilhaChecklist.getRange(nome_Checklist + 'B4:D28').getValues()
    var lista_pendecias = planilhaChecklist.getRange(nome_Checklist + 'F4:F28').getValues()
    var id = gerarID()

    var inspecao = {
      id: id,
      projeto: proj,
      data_inspecao: data,
      obsQualidade: observacao_qualidade,
      obsProjeto: observacao_projeto,
      checklist: lista_checklist,
      descricao: lista_pendecias,
      rows: lista_checklist.length
    }

    /**Registro de algumas chaves do Objeto na planilha externa. Salvos separadamente para 
     * fazer uma consulta e get mais rápido.
     */
    var linha = linhaRegistro == null ? get_linha_vazia('registros') : linhaRegistro
    planilhaExterna.getRange(nome_planExterna + 'A' + linha).setValue(id)
    planilhaExterna.getRange(nome_planExterna + 'B' + linha).setValue(proj)
    planilhaExterna.getRange(nome_planExterna + 'C' + linha).setValue(JSON.stringify(inspecao))
    planilhaExterna.getRange(nome_planExterna + 'D' + linha).setValue(data)
    planilhaExterna.getRange(nome_planExterna + 'E' + linha).setValue(userName)
    planilhaChecklist.getRange(nome_Checklist + 'C1').setValue(id)
    planilhaChecklist.getRange(nome_Checklist + 'F2').setValue(data)

    /**Variáveis globais */
    setProperty('num_proj', proj)
    setProperty('data_insp', data)
    setProperty('resetado', false)
  }
}

function get_inspObjeto(){
 if(actionCheck("Deseja carregar a inspeção?") == "YES"){
   var origem = SpreadsheetApp.getActive().getSheetByName('registros')
   var destino = SpreadsheetApp.getActive().getSheetByName('Checklist')

   /**Planilha Destino */
   const cel_id = 'C1'
   const cel_data = 'F2'
   zerar('Checklist!', false)
   var objetoInspecao;

   if (!destino.getRange(cel_id).isBlank()) {
     /**PEGA A INSPEÇÃO COM A ID ESPECIFICA LISTADA
     * 
     * Se for informado algum ID na planilha de Cchecklist, na planilha 'registros!K2'será retornado, através 
     * da fórmula "=CORRESP(ChecklistGRC!D3;A:A)", linha exata de onde consta o ID da inspeção.
     */
     objetoInspecao = JSON.parse(origem.getRange('registros!C' + origem.getRange('registros!M2').getValue()))
     transcrever(destino, objetoInspecao)

   } else if (!destino.getRange(cel_data).isBlank()) {
     /**Em 'registros!L2' existe a formula "=CORRESP(FILTER(A:A;D:D=ChecklistGRC!D4;B:B=ChecklistGRC!D2);A:A)" que filtra
      * e retorna a linha exata do ID da inspeção cujo o projeto e a data são o que constam na planilha de checklist
      */
     objetoInspecao = JSON.parse(origem.getRange('registros!C' + origem.getRange('registros!N2').getValue()).getValue())
     transcrever(destino, objetoInspecao)
   } else {
     /**Data e ID vazio => Pega data mais recente
     * 
     * Em 'registros!J2' tem a formula "=CORRESP($H$2;A:A)" que retorna a linha exata de onde está registrada a ultima inspeção
     * do projeto que consta na planilha de checklist.
     */
     objetoInspecao = JSON.parse(origem.getRange('registros!C' + origem.getRange('registros!L2').getValue()).getValue())
     transcrever(destino, objetoInspecao)
  } 
 }
}


function transcrever(planilhaDestino, objetoInspecionado){
  /**Se a planilha de Checklist for alterada (o range alterado) o range das linhas abaixo deverão ser ajustadas   */

  rows = 3 + Number(objetoInspecionado.rows)
  planilhaDestino.getRange('B4:D' + rows).setValues(objetoInspecionado.checklist)
  planilhaDestino.getRange('C1').setValue(objetoInspecionado.id)
  planilhaDestino.getRange('F2').setValue(objetoInspecionado.data_inspecao)
  planilhaDestino.getRange('F4:F' + rows).setValues(objetoInspecionado.descricao) /**Obs das pendências */
  planilhaDestino.getRange('F29').setValue(objetoInspecionado.obsProjeto)
  planilhaDestino.getRange('F30').setValue(objetoInspecionado.obsQualidade)
  

  /**Variáveis globais */
  setProperty('num_proj', objetoInspecionado.projeto)
  setProperty('data_insp', objetoInspecionado.data_inspecao)
  setProperty('resetado', false)

}


function gerarID() {
  var timestamp = new Date().getTime().toString();
  var aleatorio = Math.floor(Math.random() * 10);  // Número aleatório de 0 a 999
  var id = timestamp + aleatorio.toString();

  return id;
  // Logger.log(id)
}


function get_linha_vazia(nomePlanilha = null) {
  /**Ela não verifica se está a linha INTEIRA vazia. Apenas até a coluna necessária */
  var spreadsheet = SpreadsheetApp.getActive()
  // var spreadsheet = SpreadsheetApp.openByUrl(URL)
  var planilha = nomePlanilha == null ? spreadsheet.getSheetByName('registros') : nomePlanilha
  // var spreadsheet = SpreadsheetApp.openByUrl(URL)
  // var a = spreadsheet.getRange('registros!A1:C1').getValues()
  var linhaVazia = false;
  var i = 1;
  // var a_col = 'registros!A'; var b_col = 'registros!B'; var c_col = 'registros!C'; var d_col = 'registros!D';
  // var a_col =  planilha + '!A'; var b_col = planilha + '!B'; var c_col = planilha + '!C'; var d_col = planilha + '!D';
  // var a_col = 'GRCregistros!A'; var b_col = 'GRCregistros!B'; var c_col = 'GRCregistros!C'; var d_col = 'GRCregistros!D';
  var a_col = `${planilha}!A`; var b_col = `${planilha}!B`; var c_col = `${planilha}!C`; var d_col = `${planilha}!D`;
  Logger.log('É igual?' + (spreadsheet.getRange(c_col + 3).getValue() == ''))

  while (linhaVazia == false) {
    // linhaVazia = ((spreadsheet.getRange(a_col + i).getValue() == '') && (spreadsheet.getRange(b_col + i).getValue() == '') && (spreadsheet.getRange(c_col + i).getValue() == '') && (spreadsheet.getRange(d_col + i) == '' )) ? true : false;

    if ((spreadsheet.getRange(a_col + i).getValue() == '') && (spreadsheet.getRange(b_col + i).getValue() == '') && (spreadsheet.getRange(c_col + i).getValue() == '') && (spreadsheet.getRange(d_col + i).getValue() == '')) {
      linhaVazia = true;
    } else {
      i++;
    }
  }
  return i;
}

function prompt_proj_msg() {
  var ui = SpreadsheetApp.getUi();
  var spreadsheet = SpreadsheetApp.getActive()
  var response = ui.prompt('Projeto deve ter o código completo: nnn.cc \n Insira o código do projeto correto: ')

  if (response.getResponseText().indexOf(String.fromCharCode(46)) != -1) {
    spreadsheet.getRange('Checklist!E2').setValue(response.getResponseText())
  } else {
    prompt_proj_msg();
  }
}

function actionCheck(texto) {
  var ui = SpreadsheetApp.getUi()
  return ui.alert(texto, ui.ButtonSet.YES_NO)
}

function testePrompt() {
  var ui = SpreadsheetApp.getUi();
  var response = ui.prompt('DIGITE A SENHA: ');
  if (response.getResponseText() == 'ok') {
    ui.alert('certo!')
  }
  // Logger.log('Resposta: ' + response.getResponseText())
}


function substituir() {
  var planilhaExterna = SpreadsheetApp.openByUrl(URL);
  var planilhaRegistros = SpreadsheetApp.getActive().getSheetByName('registros')
  var linhaEncontrada = planilhaRegistros.getRange('M2').getValue()
  var userRegistrado = planilhaExterna.getRange('registros!E' + linhaEncontrada).getValue()

  /**O master pode fazer a substituição de qualquer inspeção. Se não for o master, a inspeção a ser substituída
  deve ser de autoria daquele que quer substituir.*/

  if ((userName == userRegistrado || userName == masterUserName) && (actionCheck('Tem certeza? Isso irá substituir a inspeção que consta carregada. Você só pode substituir inspeção feita por você!') == 'YES')) {
    // set_inspecao(linhaEncontrada)
    set_inspObj(linhaEncontrada)
  } else {
    actionCheck('Substituição não foi realizada.')
  }
}
// function substituir() {
//   var planilhaExterna = SpreadsheetApp.openByUrl(URL);
//   var planilhaChecklist = SpreadsheetApp.getActive()
//   var projId = planilhaChecklist.getRange('Checklist!C1').getValue()
//   var projIdBuscador = planilhaExterna.getRange('registros!A:A').createTextFinder(projId);
//   var linhaEncontrada = projIdBuscador.findAll()[0].getRow();
//   var userRegistrado = planilhaExterna.getRange('registros!E' + linhaEncontrada).getValue()

//   /**O master pode fazer a substituição de qualquer inspeção. Se não for o master, a inspeção a ser substituída
//   deve ser de autoria daquele que quer substituir.*/

//   if ((userName == userRegistrado || userName == masterUserName) && (actionCheck('Tem certeza? Isso irá substituir a inspeção que consta carregada. Você só pode substituir inspeção feita por você!') == 'YES')) {
//     set_inspecao(linhaEncontrada)
//   } else {
//     actionCheck('Substituição não foi realizada.')
//   }
// }




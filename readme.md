<div>
<img style="100%" src="https://capsule-render.vercel.app/api?type=rounded&height=100&section=footer&reversal=true&text=leIA&fontSize=70&fontColor=FFFFFF&fontAlign=50&fontAlignY=50&stroke=-&animation=twinkling&descSize=20&descAlign=50&descAlignY=50&theme=radical" />
</div>
<h1 align="left">Documentação de Fluxos e Steps</h1>
<h2 align="center">FLUXO: ONBOARDING</h2>
<h5 align="left">O fluxo de onboarding é a porta de entrada para novos usuários ou usuários não identificados. Ele garante que a LGPD seja respeitada e tenta localizar o condutor na base de dados através do CPF.</h5>
<h6 align="left">ONBOARDING - CONSENT:</h6>
<p align="left">Esta é a primeira etapa do fluxo. Seu objetivo é obter o consentimento explícito do cliente para o uso e a consulta de seus dados pessoais, conforme a LGPD.<br>
<b>Mensagem:</b> "Olá 👋 Sou a Léia... Você autoriza a consulta e o uso dos seus dados pessoais para prosseguir? SIM | NÃO"<br>
<b>Resultado:</b> SIM (Segue para CPF) | NÃO (Finaliza atendimento).</p>
<h6 align="left">ONBOARDING - CPF:</h6>
<p align="left">Segunda etapa do onboarding. Após o consentimento, o sistema solicita o CPF para verificar se o condutor já possui cadastro ativo no sistema Radar Logístico.<br>
<b>Lógica:</b> O sistema valida se o CPF possui 11 dígitos e aplica o cálculo de dígito verificador. Se o CPF for válido, o sistema faz uma consulta via API externa (Radar Logístico).<br>
<b>Resultado esperado:</b><br>
- <b>Condutor Encontrado:</b> O sistema direciona o usuário diretamente para o <b>MENU</b> principal.<br>
- <b>Condutor Não Encontrado:</b> O sistema direciona para a etapa de <b>REGISTRATION_RESPONSE</b>.</p>
<h6 align="left">ONBOARDING - REGISTRATION_RESPONSE:</h6>
<p align="left">Etapa de decisão para usuários não localizados. Oferece ao usuário a chance de iniciar um cadastro do zero ou tentar digitar o CPF novamente.<br>
<b>Mensagem:</b> "Sinto muito, mas não encontrei o seu cadastro... O que me diz? 1 - Realizar cadastro | 2 - Tentar Novamente | 3 - Finalizar"<br>
<b>Resultado esperado:</b><br>
- <b>Opção 1:</b> Altera o fluxo para <b>REGISTER</b>.<br>
- <b>Opção 2:</b> Retorna ao step <b>CPF</b>.<br>
- <b>Opção 3:</b> Finaliza o atendimento.</p>
<h2 align="center">FLUXO: REGISTER (Cadastro de Condutor)</h2>
<h5 align="left">Este fluxo é acionado quando um novo motorista precisa ser inserido no sistema. Ele é focado na coleta de documentos (fotos) e validação de informações essenciais para o convite (invite) de cadastro.</h5>
<h6 align="left">REGISTER - CNH:</h6>
<p align="left">Solicita ao usuário uma foto nítida da Carteira Nacional de Habilitação.<br>
<b>Lógica:</b> O nó de validação verifica se o arquivo enviado é uma URL de imagem válida. Em caso positivo, o link é armazenado na tabela <code>cnh_url</code> associada à sessão.</p>
<h6 align="left">REGISTER - CRLV_EXPECTED_RESPONSE:</h6>
<p align="left">O sistema pergunta ao motorista quantos veículos (CRLV) ele deseja cadastrar.<br>
<b>Lógica:</b> O sistema aceita valores numéricos (geralmente de 1 a 5). Este número define quantas vezes o próximo loop de captura de imagem será repetido.</p>
<h6 align="left">REGISTER - CRLV_LOOP:</h6>
<p align="left">Etapa iterativa onde o sistema solicita as fotos dos CRLVs.<br>
<b>Lógica:</b> A cada imagem enviada, o sistema consulta no banco (Postgres) quantos arquivos já foram recebidos vs. o esperado. Enquanto o número não for atingido, ele continua solicitando "Mande a foto do CRLV X/Y".</p>
<h6 align="left">REGISTER - ANTT_EXPECTED_RESPONSE:</h6>
<p align="left">Similar ao CRLV, questiona a quantidade de certificados ANTT que o motorista possui para realizar a carga dos arquivos.</p>
<h6 align="left">REGISTER - ANTT_LOOP:</h6>
<p align="left">Captura as imagens das ANTTs informadas.<br>
<b>Lógica:</b> Armazena os links na tabela <code>antt_files</code> e controla a contagem de recebimento.</p>
<h6 align="left">REGISTER - COMP_RESID:</h6>
<p align="left">Última etapa de coleta de documentos, solicitando o Comprovante de Residência.<br>
<b>Finalização:</b> Após o recebimento deste arquivo, o sistema consolida todos os links (CNH, CRLVs, ANTTs, Residência) em um JSON estruturado e dispara uma mensagem de sucesso, movendo o usuário para o fluxo de <b>MENU</b>.</p>
<h2 align="center">FLUXO: MENU</h2>
<h5 align="left">O Menu é o hub central de comandos para motoristas já cadastrados. Ele permite a navegação entre as funcionalidades operacionais do sistema.</h5>
<h6 align="left">MENU - MENU_RESPONSE:</h6>
<p align="left">Apresenta as opções disponíveis para o motorista.<br>
<b>Opções:</b><br>
- <b>[1] Atualizar Dados:</b> Encaminha para o fluxo <b>UPDATE_DATA</b>.<br>
- <b>[2] Ver Coletas Disponíveis:</b> Encaminha para o fluxo <b>AVAILABLE_COLLECTIONS</b>.<br>
- <b>[3] Continuar Viagem Atual:</b> Encaminha para o fluxo <b>TRIP</b>.<br>
- <b>[4] Finalizar/Sair:</b> Encerra a sessão atual.</p>


import jade.core.AID;
import jade.core.Agent;
import jade.core.behaviours.CyclicBehaviour;
import jade.domain.DFService;
import jade.domain.FIPAException;
import jade.domain.FIPAAgentManagement.DFAgentDescription;
import jade.domain.FIPAAgentManagement.ServiceDescription;
import jade.lang.acl.ACLMessage;
import jade.lang.acl.UnreadableException;

// Parametros de inicialização no ambiente eclipse(Run Configurations --> Arguments)
// Mestrado: -gui mercado:AgenteMercado;os:AgenteOS;tucurui:AgenteUsina(100,Tucurui,0,5000,50,4000);BeloMonte:AgenteUsina(80,BeloMonte,0,8000,50,5000);Urucum:AgenteUsina(200,Urucum,0,8000,20,3000);
// Teste Futuro: -gui Sudeste:AgenteMercado;ons:AgenteOS;tucurui:AgenteUsina(100,Tucurui,0,5000,50,4000);BeloMonte:AgenteUsina(80,BeloMonte,0,8000,50,5000);

public class AgenteOS extends Agent{

	private static final long serialVersionUID = 1L;		
	
	
	protected void setup(){
		
		//-------------  Registrando Agente no DF
		DFAgentDescription dfd = new DFAgentDescription();
		dfd.setName(getAID());
		ServiceDescription sd = new ServiceDescription();
		sd.setName("operador-do-sistema");
		sd.setType("operador-do-sistema");
		dfd.addServices(sd);
		try{
			DFService.register(this, dfd);
		}
		catch(FIPAException fe){
			fe.printStackTrace();
		}
		
		System.out.println("----> Agente <"+getLocalName()+">: \n" +
				"           Agente Iniciado e Serviço Registrado no DF");

		
		
//##### ----------  comportamentos ---------- #####
		
		addBehaviour(new OSRecebeDados());
		//Capturando estado do agente
		//System.out.println("==== Estado do Agente OS: "+this.getAgentState());
		
	} //fim Setup

	
	

	
	
	
	
//###### --------- Classes do OS -----#######
	
	
	
	public class OSRecebeDados extends CyclicBehaviour {
		static final long serialVersionUID = 1L;
		
		int contadorRecebeValorUsina = 0;
		int contadorPeriodo = 0;
		int contadorPrecoComercializado = 0;
		double soma = 0;
		
		Double[] recebeValorUsina;
		Double[] periodo;
		String[] vetorPrecoComercializado;
		
		DFAgentDescription[] result;
		AID[] endServico;
		ACLMessage msg;
		
		
		
			
		
		
		public void action(){
			
		
			msg = blockingReceive();
			if(msg == null){
				
				block();
				return;
			}
			
			
			if(msg.getPerformative() == ACLMessage.INFORM && msg.getOntology() == "Mercado"){
				
				/*
				//Analisa mensagem recebida do mercado com os preços do período analisado
				analisaMensagem();
				*/
				
				try {
					
					vetorPrecoComercializado =  (String[]) msg.getContentObject();
					//CRIA VETOR DE PERÍODO
					periodo = new Double[vetorPrecoComercializado.length];
					for(int i=0; i<vetorPrecoComercializado.length; i++){  
						System.out.println("----------------------------------------- \n" +
								"> AgenteOS recebe "+vetorPrecoComercializado.length+" valores sendo os Precos do mercado\n" +
								"Preco Spot no mês " + (i+1) + " = " +vetorPrecoComercializado[i] +"\n" +
										"============================================="); 	            			            
			        }
				} catch (UnreadableException e) {
					
					e.printStackTrace();
				}								
				
				//BUSCANDO USINAS
				System.out.println("---->Agente <"+ getAID().getLocalName()+">:\n" +
						"           Buscando Agente Usina \n");										
				
				DFAgentDescription template = new DFAgentDescription();
				ServiceDescription sds = new ServiceDescription();
				String servico = "Usina";
				sds.setName(servico);
				sds.setType(servico);
				template.addServices(sds);
				
				try{
					
					result = DFService.search(myAgent, template);//result é um vetor de descriçaõ de serviços dos agentes			
					endServico = new AID[result.length];	//endService é um vetor que contem os agentes com serviços do encontrados no vetor result

								
					System.out.println("---->Agente <"+ getAID().getLocalName()+">:\n" +
							"           "
							+result.length+" AGENTE(S) ENCONTRADO(S) ....\n" +
									"Os Agentes encontrados são:");	
		            
					for (int i = 0; i < result.length; ++i) {
		            	endServico[i] = result[i].getName();
		            	System.out.println(endServico[i].getLocalName());		            			            	
		            }
		            
					//CRIA UM VETOR PARA RECEBER OS VALORES ENTREGUES PELAS USINAS
					recebeValorUsina = new Double[result.length];
		            
				}
				catch (FIPAException fe) {
		            fe.printStackTrace();
		            
		        } 
				
				//ENVIANDO PREÇO COMERCIALIZADO
				enviaPrecoComercializado();
				/*
				String m = vetorPrecoComercializado[contadorPrecoComercializado];
				for (int i = 0; i < result.length; ++i) {
					System.out.println("CONTADOR DO PREÇO COMERCIALIZADO: "+contadorPrecoComercializado);
	            	endServico[i] = result[i].getName();       		       		            	
	            	msg.setPerformative(ACLMessage.REQUEST);
					msg.setSender(myAgent.getAID());
					msg.setOntology("Energia");
					msg.setContent(m);
					msg.addReceiver(new AID(endServico[i].getLocalName(), AID.ISLOCALNAME));
					send(msg);												
					System.out.println("MSG ENVIADA PARA: "+endServico[i].getLocalName()+" CONTEUDO: "+msg.getContent());
					//analisaMensagem();
									
	            }
				contadorPrecoComercializado++;
					*/		
				
			
			}
			
			
			// OS recebe valor de cada usina e calcula o total para entregar ao mercado
			if(msg.getPerformative() == ACLMessage.INFORM && msg.getOntology() == "Usina"){			
				
				
				
				System.out.println("Contador das Mensagens recebidas: "+contadorRecebeValorUsina+" " +myAgent.getLocalName()+": Recebe mensagem de "+msg.getSender().getLocalName()+"Conteúdo da Mensgem: "+msg.getContent());
				recebeValorUsina[contadorRecebeValorUsina] = Double.parseDouble(msg.getContent());//vetor do tamanho da qtd de agentes usinas usado para receber o valor vindo das usinas														
				//System.out.println("##########################  -----------Posicao: "+contadorRecebeValorUsina+" do vetor recebeValorUsina "+recebeValorUsina[contadorRecebeValorUsina]);
				soma+=recebeValorUsina[contadorRecebeValorUsina];
				//System.out.println("SOMA recebe: "+soma+ " com Contador em: "+contadorRecebeValorUsina);
				contadorRecebeValorUsina++;
				block();
				
				//enviaPrecoComercializado();//teste
				
				/*if (contadorRecebeValorUsina == result.length ){//quando receber o valor de todos os agentes soma o periodo
					
					//periodo[contadorPeriodo] = soma;
					//System.out.println("##########################  -----------Entrega Mercado: "+periodo[contadorPeriodo]);

				
					periodo[contadorPeriodo]=soma;
					System.out.println("##########################  -----------VALOR TOTAL ENTREGUE NO MES:"+contadorPeriodo+ " é "+periodo[contadorPeriodo]);
					
					contadorPeriodo++;
					
					soma = 0;
					contadorRecebeValorUsina=0;
					//recebeValorUsina = null;
					enviaPrecoComercializado();
					//System.out.println("######### contadorPrecoComercializado "+contadorPrecoComercializado);
					//envia segundo preço do período para os agentes;
					if(contadorPrecoComercializado <= periodo.length){
						contadorRecebeValorUsina=0;
						//recebeValorUsina = null;
						enviaPrecoComercializado();
					} else{
						System.out.println("*************------------  Valor periodo calculado    ---------*********************");
				   }
						
					//doWait();
				 }*/
				
			}	
			
			//public double despacho(){
				
				//return soma;
			//}
			
			
			
	} 
		//System.out.println("##########################  -----------O Despacho entregue é: "+somaDespachoUsinas);		
		
		
		
		public void buscaAgentes(){
			//BUSCANDO AGENTES USINAS NO SISTEMA
					System.out.println("---->Agente <"+ getAID().getLocalName()+">:\n" +
							"           Buscando Agente Usina \n");										
					
					DFAgentDescription template = new DFAgentDescription();
					ServiceDescription sds = new ServiceDescription();
					String servico = "Usina";
					sds.setName(servico);
					sds.setType(servico);
					template.addServices(sds);
					
					try{
						
						result = DFService.search(myAgent, template);//result é um vetor de descriçaõ de serviços dos agentes			
						endServico = new AID[result.length];	//endService é um vetor que contem os agentes com serviços do encontrados no vetor result

									
						System.out.println("---->Agente <"+ getAID().getLocalName()+">:\n" +
								"           "
								+result.length+" AGENTE(S) ENCONTRADO(S) ....\n" +
										"Os Agentes encontrados são:");	
			            
						for (int i = 0; i < result.length; ++i) {
			            	endServico[i] = result[i].getName();
			            	System.out.println(endServico[i].getLocalName());		            			            	
			            }
			            
						//criando vetor para somar entregas das usinas
						recebeValorUsina = new Double[result.length];
			            
					}
					catch (FIPAException fe) {
			            fe.printStackTrace();
			            
			        } 
		}
		
		
		
		public void enviaPrecoComercializado(){
			String m = vetorPrecoComercializado[contadorPrecoComercializado];//pega os valores um a um dos preços comercializados e envia para todas as usinas
			for (int i = 0; i < result.length; ++i) {
				System.out.println("CONTADOR DO PREÇO COMERCIALIZADO--> pega valor do preço comercializado: "+contadorPrecoComercializado);
            	endServico[i] = result[i].getName();       		       		            	
            	msg.setPerformative(ACLMessage.REQUEST);
				msg.setSender(myAgent.getAID());
				msg.setOntology("Energia");
				msg.setContent(m);
				msg.addReceiver(new AID(endServico[i].getLocalName(), AID.ISLOCALNAME));
				send(msg);												
				System.out.println("MSG ENVIADA PARA: "+endServico[i].getLocalName()+" CONTEUDO: "+msg.getContent());
				//analisaMensagem();
								
            }
			contadorPrecoComercializado++;
		}
		
		public void enviaPrecoComercializado2(){
			
			
			String m = vetorPrecoComercializado[contadorPrecoComercializado];
			for (int i = 0; i < result.length; ++i) {
            	endServico[i] = result[i].getName();       		       		            	
            	msg.setPerformative(ACLMessage.REQUEST);
				msg.setSender(myAgent.getAID());
				msg.setOntology("Energia");
				msg.setContent(m);
				msg.addReceiver(new AID(endServico[i].getLocalName(), AID.ISLOCALNAME));
				send(msg);												
				analisaMensagem();
								
            }
			contadorPrecoComercializado++;
			
			
		}
		
		public void somaEnergiaAgentes(){
			//"entrega" é um vetor criado no método buscaAgentes, do tamanho da qtd de agentes usinas do sistema
			//sErve para somar os valores de entrega dos agentes usinas
			recebeValorUsina[contadorRecebeValorUsina] = Double.parseDouble(msg.getContent());
			soma+=recebeValorUsina[contadorRecebeValorUsina];//soma os valores do vindos de todos os agentes usina
			contadorRecebeValorUsina++;
			
				if (contadorRecebeValorUsina == recebeValorUsina.length ){
					
					periodo[contadorPeriodo] = soma;
					
					System.out.println("##########################  -----------Entrega Mercado: "+periodo[contadorPeriodo]);
					
					contadorPeriodo++;
					
					//envia segundo preço do período para os agentes;
					//enviaPrecoComercializado2();
					
					block();
				}
				
				
		}
		
		public void analisaMensagem(){
			
			System.out.println("-------------- "+myAgent.getLocalName()+": Analisando a msg ------ \n" +
					"           Sender: <"+msg.getSender().getLocalName()+"\n" +
					"           Performativa: "+ msg.getPerformative()+"\n" +
					"           Ontologoia: "+msg.getOntology()+"\n" +
					"           Conteúdo: "+msg.getContent()+"\n"  +
					"====================================================");
		
			
			
		}
		
		protected void takeDown(){
			doDelete();		
		}
	
	}
	
	
	
}
	


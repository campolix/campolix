import jade.core.Agent;
import jade.core.behaviours.CyclicBehaviour;
import jade.core.behaviours.OneShotBehaviour;
import jade.domain.DFService;
import jade.domain.FIPAException;
import jade.domain.FIPAAgentManagement.DFAgentDescription;
import jade.domain.FIPAAgentManagement.ServiceDescription;
import jade.lang.acl.ACLMessage;
import br.edu.ufg.master.base.Usina;
//beloMonte:AgenteUsina(100,BeloMonte,0,4000,2000,6000);

public class AgenteUsina extends Agent{
	private static final long serialVersionUID = 1L;
	
	private String nome;
	private String volumeMin;
	private String volumeMax;
	private String geracaoMin;
	private String geracaoMax;
	private String precoEnergia;
	
	
	
	private String despacho;
	private double precoMercado;
	private double geracao = 0;
	private double precoEnergiaUsina;
	
	//private double[] afluencias = new double[6];
	
	//private String operacional;
	//private String naoOperacional;
	String status;
	Usina u;
	
	
	protected void setup(){
		
		System.out.println("--AgUsina--> Agente <"+getLocalName()+">: \n" +
				"           Agente Iniciado e Serviço Registrado no DF");
		//---------------Instanciando o Agente do tipo Usina
		u = new Usina();
			//Capturando dados de entrada do agente
			//Formato de entrada: (double precoEnergia, String nome, double volumeMin, double volumeMax, double geracaoMin, double geracaoMax)
			Object[] args = getArguments();
			if(args != null && args.length>0){
				precoEnergia = (String) args[0];
				nome = (String) args[1];
				volumeMin = (String) args[2];
				volumeMax = (String) args[3];
				geracaoMin = (String) args[4];
				geracaoMax = (String) args[5];
				
				
				// Populando dados do agente
				u.setPrecoEnergia(Double.parseDouble(precoEnergia));
				u.setNome(nome);
				u.setVolumeMin(Double.parseDouble(volumeMin));
				u.setVolumeMax(Double.parseDouble(volumeMax));
				u.setGeracaoMin(Double.parseDouble(geracaoMin));
				u.setGeracaoMax(Double.parseDouble(geracaoMax));
				/*	
				System.out.println("Nome da Usina: "+u.getNome());
				System.out.println("Preço da Energia: "+u.getPrecoEnergia());
				System.out.println("Volume Min: "+u.getVolumeMin());
				System.out.println("Volume Max: "+u.getVolumeMax());
				System.out.println("Geracao Min: "+u.getGeracaoMin());
				System.out.println("Geracao Max: "+u.getGeracaoMax());
				*/
			} else{
				System.out.println("fim");
				doDelete();
			}
				
			//-------------Registrando os serviços deste agente no DF(páginas amarelas
			DFAgentDescription dfd =  new DFAgentDescription();
			dfd.setName(getAID());
			ServiceDescription sd = new ServiceDescription();
			sd.setType("usina");
			sd.setName("usina");
			dfd.addServices(sd);
			
			
			
			try{
				DFService.register(this, dfd);
			}
			catch(FIPAException e){
				e.printStackTrace();
			}
			
			
			
			
//------------   Comportamentos do Agente Usina --------------------------

			
			addBehaviour(new RecebeDadosOS()); 
				
			//System.out.println("##########################  -----------O estado do agente é: "+getAgentState());
				
	}	//fim do método setup do Agente Usina			
	
	protected void takeDown(){

		try{ DFService.deregister(this); }
		catch(FIPAException e){
			e.printStackTrace();
		}
		System.out.println("--AgUsina--> Agente Morreu");
	}
	
// -------------------- Classes dos comportamentos
	private class RecebeDadosOS extends CyclicBehaviour{

	private static final long serialVersionUID = 1L;

	
	public void action() {
		
		System.out.println("Agente "+getLocalName()+":\n" +
				"            waiting for REQUEST message...--AgUsina-->");
		ACLMessage msg = blockingReceive();
		
		if (msg == null){
			block();
			return;
		}
		if (msg.getPerformative()==ACLMessage.REQUEST){
			
			System.out.println("---->Agente <"+ getAID().getLocalName()+">:\n" +
					"           RECEBE REQUEST do agente: <"+msg.getSender().getLocalName()+"\n" +
					"           A performativa da msg é: "+ msg.getPerformative()+"\n" +
					"           A ontologoia da mensagem é: "+msg.getOntology()+"\n" +
					"           O conteúdo da mensagem é: "+msg.getContent());
			try{							
			//Algoritimo para definir despacho			
			precoMercado = Double.parseDouble(msg.getContent());
			precoEnergiaUsina = u.getPrecoEnergia();
			
			
			
			if(precoEnergiaUsina >= precoMercado){
				 geracao = u.getGeracaoMin();
				 /*System.out.println("--AgUsina--> Agente <"+getLocalName()+"\n" +
						"          Preco Energia da Usina: "+precoEnergiaUsina+"\n" +
						"          Preço do Mercado Comercializado : "+precoMercado+"\n"+
				 		"          Decisão de Geração Minima: "+geracao);*/
			} else if (precoEnergiaUsina < precoMercado){
				geracao = u.getGeracaoMax();
				/*System.out.println("--AgUsina--> Agente <"+getLocalName()+"\n" +
						"          Preco Energia da Usina: "+precoEnergiaUsina+"\n" +
						"          Preço do Mercado Comercializado : "+precoMercado+"\n"+
				 		"          Decisão de Geração Maxima: "+geracao);*/
				
			}
			
			despacho = String.valueOf(geracao);
			
			//Resposta ao OS do despacho possível			
			ACLMessage reply = msg.createReply();
			reply.setPerformative(ACLMessage.INFORM);
			reply.setOntology("Usina");
			reply.setContent(despacho);
			//myAgent.send(reply);
			send(reply);
			
			System.out.println("---->Agente <"+ getAID().getLocalName()+">:\n" +
					"           RESPONDE COM: \n" +
					"           A nova performativa da msg é: "+ reply.getPerformative()+"\n" +
					"           A ontologoia da mensagem é: "+reply.getOntology()+"\n" +
					"           O conteúdo da mensagem é: "+reply.getContent());
			
			
			} catch (Exception e) {
				e.printStackTrace(); 
			} 
		}
		
	}
	
	}
	
}

////------ Códigos para aproveitamento
/*/----Registrar Agente nas páginas amarelas(DF)
//Criando uma entrada no DF
DFAgentDescription dfd = new DFAgentDescription();
dfd.setName(getAID());

//Criando um serviço
ServiceDescription sd = new ServiceDescription();
sd.setType("Despacho");
sd.setName("preco");


//adicionando o serviço df DF
dfd.addServices(sd);

//Registrando Agente no DF
try{
	//regitro(Agente que oferece, descrição)
	DFService.register(this, dfd);
}catch(FIPAException e){
	e.printStackTrace();
}
*///-------- fim registro
		
/*// Inicio	class UsinaInformaStatusOS teste
private class UsinaInformaStatusOS extends OneShotBehaviour{
			
	
	private static final long serialVersionUID = 1L;

	public void action(){
			
		if(AgenteUsina.u.getGeracaoMax() <= u.getVolumeMax()){
				
			//System.out.println("Volume Máximo "+u.getVolumeMax());
			//System.out.println("Geração Máxima "+u.getGeracaoMax());					
			
			//String conteudo = u.getPrecoEnergia();
			//Double precoEnergia = u.getPrecoEnergia();
			//String conteudo;
			//conteudo = String.valueOf(precoEnergia); 
			
			ACLMessage msg = new ACLMessage(ACLMessage.INFORM);
			msg.addReceiver(new AID("OS", AID.ISLOCALNAME));
			msg.setLanguage("Portugues");
			msg.setOntology("Dados de Usina");
			msg.setContent(conteudo);
			myAgent.send(msg);
			System.out.println(" Agente <"+ getAID().getLocalName()+"> enviou mensagem "+msg.getContent());
			//System.out.println("msg "+msg.getContent()+" enviada");
			//System.out.println(status);

			}else{
				
				status = naoOperacional;
				ACLMessage msg = new ACLMessage(ACLMessage.INFORM);
				msg.addReceiver(new AID("OS", AID.ISLOCALNAME));
				msg.setLanguage("Portugues");
				msg.setOntology("Demanda-de-Mercado");
				msg.setContent("naoOperacional");
				myAgent.send(msg);
				System.out.println(" Agente <"+ getAID().getLocalName()+"> enviou mensagem "+msg.getContent());

				//System.out.println(getAID().getLocalName()+" --> Mensagem "+msg.getContent()+" enviada");
				//System.out.println(status);
			}
		
		}
	}	
			
}*///fim classe teste usinaInfomaStatus

/*
private class UsinaInformaPrecoOS extends OneShotBehaviour{

	private static final long serialVersionUID = 1L;

	public void action(){
		
		String conteudo = String.valueOf(u.getPrecoEnergia());
		ACLMessage msg = new ACLMessage(ACLMessage.INFORM);
		msg.addReceiver(new AID("OS", AID.ISLOCALNAME));
		msg.setLanguage("Portugues");
		msg.setOntology("Dados de Usina");
		msg.setContent(conteudo);
		send(msg);
		//System.out.println("--AgUsi--> O Agente "+msg.getSender().getLocalName()+ " enviou  seu preço para OS R$"+msg.getContent());
	}
	
}


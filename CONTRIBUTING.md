adicionei ao projeto localmente

//#src/main/java/br/ufsc/bridge/mpiclient/model/CNS.java

package br.ufsc.bridge.mpiclient.model;

import lombok.*;

import br.ufsc.bridge.metafy.Metafy;

@Metafy
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
public class CNS {
	private String numero;
	private String tipo;
}





//#src/main/java/br/ufsc/bridge/mpiclient/responsehandler/pdq/CNSHandler.java

package br.ufsc.bridge.mpiclient.responsehandler.pdq;

import br.ufsc.bridge.mpiclient.util.XmlAttributesWrapper;
import lombok.NonNull;

import org.xml.sax.Attributes;
import org.xml.sax.ContentHandler;
import org.xml.sax.SAXException;
import org.xml.sax.XMLReader;

import br.ufsc.bridge.mpiclient.model.CNS;
import br.ufsc.bridge.mpiclient.model.Cidadao;

public class CNSHandler extends DocumentHandler {

	private static final String EXTENSION = "extension";
	private CNS cns;

	public CNSHandler() {
		super("2.16.840.1.113883.13.236");
	}


	@Override
	protected void noHandlerFound(String uri, String localName, String qName, Attributes attributes) {
		XmlAttributesWrapper wrapper = XmlAttributesWrapper.wrap(attributes);
		String root = wrapper.get("root");
		if ("2.16.840.1.113883.13.236.1".equals(root)) {
			this.cns.setTipo(wrapper.get(EXTENSION));
		}
	}

	@Override
	protected void returnToParent(String uri, String localName, String qName) throws SAXException {
		current.getCnss().add(this.cns);
		this.cns = null;
		super.returnToParent(uri, localName, qName);
	}

	public void handle(@NonNull XMLReader reader, @NonNull ContentHandler parent, @NonNull Cidadao current, @NonNull Attributes attr) {
		super.handle(reader, parent, current, attr);
		this.cns = new CNS();
		this.cns.setNumero(attr.getValue(EXTENSION));
	}
}


//#src/test/java/br/ufsc/bridge/mpiclient/messages/PRPA_IN201301UV02Test.java

package br.ufsc.bridge.mpiclient.messages;

import static org.assertj.core.api.Assertions.assertThat;

import java.io.IOException;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

import org.junit.Test;
import org.xml.sax.SAXException;

import br.ufsc.bridge.mpiclient.exceptions.MPIValidationException;
import br.ufsc.bridge.mpiclient.loader.FileLoader;
import br.ufsc.bridge.mpiclient.model.CNH;
import br.ufsc.bridge.mpiclient.model.CNS;
import br.ufsc.bridge.mpiclient.model.CTPS;
import br.ufsc.bridge.mpiclient.model.CertidaoAntiga;
import br.ufsc.bridge.mpiclient.model.CertidaoNova;
import br.ufsc.bridge.mpiclient.model.Cidadao;
import br.ufsc.bridge.mpiclient.model.Contato;
import br.ufsc.bridge.mpiclient.model.Endereco;
import br.ufsc.bridge.mpiclient.model.IdentificadorLocal;
import br.ufsc.bridge.mpiclient.model.Naturalizado;
import br.ufsc.bridge.mpiclient.model.Passaporte;
import br.ufsc.bridge.mpiclient.model.RG;
import br.ufsc.bridge.mpiclient.model.TituloEleitor;
import br.ufsc.bridge.mpiclient.model.dominio.Etnia;
import br.ufsc.bridge.mpiclient.model.dominio.Pais;
import br.ufsc.bridge.mpiclient.model.dominio.RacaCor;
import br.ufsc.bridge.mpiclient.model.dominio.Sexo;
import br.ufsc.bridge.mpiclient.model.dominio.TipoCertidaoAntiga;
import br.ufsc.bridge.mpiclient.model.dominio.TipoCertidaoNova;
import br.ufsc.bridge.mpiclient.model.dominio.TipoConfidencialidade;
import br.ufsc.bridge.mpiclient.model.dominio.TipoContato;
import br.ufsc.bridge.mpiclient.model.dominio.TipoEndereco;
import br.ufsc.bridge.mpiclient.model.dominio.TipoLogradouro;
import br.ufsc.bridge.mpiclient.model.dominio.UF;

public class PRPA_IN201301UV02Test {

	@Test
	public void pedroMoreiraLauro() throws MPIValidationException, IOException, SAXException {
		Cidadao cidadao = Cidadao.builder()
				.certidao(new CertidaoAntiga(
						"NOMECARTORIO1",
						LocalDate.of(2005, 05, 05),
						"FOL1",
						"LIVRO1",
						"TERMO1",
						TipoCertidaoAntiga.CERTIDAO_DE_CASAMENTO))
				.certidao(new CertidaoNova(
						"12345678901234567890123456789012",
						LocalDate.of(2006, 06, 06),
						TipoCertidaoNova.CERTIDAO_DE_NASCIMENTO))
				.cnh(new CNH(LocalDate.of(2002, 02, 02), "789654987", UF.GOIAS))
				.cns(new CNS("111111111111111","D"))
				.cns(new CNS("888888888888888","P"))
				.contato(new Contato("+55-61-82997451", TipoContato.TELEFONE_RESIDENCIAL))
				.contato(new Contato("foo@gmail.com", TipoContato.EMAIL))
				.contato(new Contato("bar@hotmail.com", TipoContato.EMAIL))
				.cpf("01030433143")
				.ctps(new CTPS(LocalDate.of(2004, 04, 04), "1234", "123"))
				.dataNascimento(LocalDate.of(1985, 02, 05))
				.endereco(Endereco.builder()
						.bairro("Bairro")
						.cep("88888080")
						.codigoMunicipio("123456")
						.complemento("Complemento")
						.logradouro("Logradouro")
						.numero("12")
						.pais(Pais.BRASIL)
						.tipoEndereco(TipoEndereco.CASA)
						.tipoLogradouro(TipoLogradouro.RUA)
						.uf(UF.DISTRITO_FEDERAL)
						.build())
				.etnia(Etnia.TAPUIA_TAPUIA_XAVANTE_TAPUIO)
				.identificadorLocal(new IdentificadorLocal("localID", "systemCode", "oidSystemCode"))
				.nacionalidade(new Naturalizado(LocalDate.of(1997, 01, 01), LocalDate.of(1997, 02, 02), "52146546"))
				.nome("PEDRO MOREIRA LAURO")
				.nomeMae("MARIA DO CARMO MOREIRA LAURO")
				.nomePai("PAULO PINTO RIBEIRO")
				.nomeSocial("ORDEP")
				.numeroDnv("DNVNUMBER1")
				.passaporte(new Passaporte(
						LocalDate.of(2001, 01, 01),
						LocalDate.of(2015, 01, 01),
						"FB31241",
						Pais.BRASIL))
				.numeroNisPisPasep("01030433140")
				.numeroRic("RICNUMBER1")
				.racaCor(RacaCor.BRANCA)
				.rg(new RG(LocalDate.of(2003, 03, 03), "RGNUMBER1", "10", UF.DISTRITO_FEDERAL))
				.sexo(Sexo.MASCULINO)
				.tipoConfidencialidade(TipoConfidencialidade.RESTRITO)
				.tituloEleitor(new TituloEleitor("TITULONUMBER1", "Z1", "SECAO1"))
				.vivo(false)
				.build();

		String messageXml = new PIXRequestMessage().create(cidadao, DateTimeFormatter.ofPattern("yyyyMMddHHmmss").parse("20101123115812", LocalDateTime::from));

		assertThat(messageXml)
				.isNotBlank()
				.isEqualTo(FileLoader.loadAsString("/requests/PedroMoreiraLauro.xml"));
	}

}


//#src/test/java/br/ufsc/bridge/mpiclient/messages/PRPA_IN201306UV02Test.java

package br.ufsc.bridge.mpiclient.messages;

import static br.ufsc.bridge.mpiclient.model.MCidadao.meta;
import static org.assertj.core.api.Assertions.assertThat;

import java.io.InputStream;
import java.time.LocalDate;
import java.util.List;

import org.junit.Test;

import br.ufsc.bridge.mpiclient.exceptions.MPIXmlParseException;
import br.ufsc.bridge.mpiclient.model.CNS;
import br.ufsc.bridge.mpiclient.model.Cidadao;
import br.ufsc.bridge.mpiclient.model.IdentificadorLocal;
import br.ufsc.bridge.mpiclient.model.MBrasileiro;
import br.ufsc.bridge.mpiclient.model.MEndereco;
import br.ufsc.bridge.mpiclient.model.MRG;
import br.ufsc.bridge.mpiclient.model.dominio.Pais;
import br.ufsc.bridge.mpiclient.model.dominio.RacaCor;
import br.ufsc.bridge.mpiclient.model.dominio.Sexo;
import br.ufsc.bridge.mpiclient.model.dominio.TipoConfidencialidade;
import br.ufsc.bridge.mpiclient.model.dominio.TipoEndereco;
import br.ufsc.bridge.mpiclient.model.dominio.TipoLogradouro;
import br.ufsc.bridge.mpiclient.model.dominio.UF;

public class PRPA_IN201306UV02Test {

	@Test
	public void severinoFaustino() throws MPIXmlParseException {
		InputStream stream = this.getClass().getResourceAsStream("/responses/SeverinoFaustino.xml");
		List<Cidadao> read = new PDQResponseMessage().read(stream);

		assertThat(read).hasSize(1);

		Cidadao cidadao = read.get(0);

		assertThat(cidadao)
				.hasFieldOrPropertyWithValue(meta.dataNascimento.getAlias(), LocalDate.of(1938, 06, 25))
				.hasFieldOrPropertyWithValue(meta.nome.getAlias(), "SEVERINO FAUSTINO")
				.hasFieldOrPropertyWithValue(meta.nomeMae.getAlias(), "AMALIA COELHO DE MELO")
				.hasFieldOrPropertyWithValue(meta.nomePai.getAlias(), "SEM INFORMAÇÃO")
				.hasFieldOrPropertyWithValue(meta.racaCor.getAlias(), RacaCor.PRETA)
				.hasFieldOrPropertyWithValue(meta.sexo.getAlias(), Sexo.MASCULINO)
				.hasFieldOrPropertyWithValue(meta.tipoConfidencialidade.getAlias(), TipoConfidencialidade.NORMAL);

		assertThat(cidadao.getCnss())
				.containsOnly(new CNS("898002940850595","D"));

		assertThat(cidadao.getEnderecos().get(0))
				.hasFieldOrPropertyWithValue(MEndereco.meta.bairro.getAlias(), "CIDADE DE DEUS")
				.hasFieldOrPropertyWithValue(MEndereco.meta.cep.getAlias(), "22763550")
				.hasFieldOrPropertyWithValue(MEndereco.meta.codigoMunicipio.getAlias(), "330455")
				.hasFieldOrPropertyWithValue(MEndereco.meta.complemento.getAlias(), "FDS CASA")
				.hasFieldOrPropertyWithValue(MEndereco.meta.logradouro.getAlias(), "DA BIBLIA")
				.hasFieldOrPropertyWithValue(MEndereco.meta.numero.getAlias(), "64")
				.hasFieldOrPropertyWithValue(MEndereco.meta.pais.getAlias(), Pais.BRASIL)
				.hasFieldOrPropertyWithValue(MEndereco.meta.tipoEndereco.getAlias(), TipoEndereco.CASA)
				.hasFieldOrPropertyWithValue(MEndereco.meta.tipoLogradouro.getAlias(), TipoLogradouro.PRACA);

		assertThat(cidadao.getIdentificadoresLocais())
				.containsOnly(
						new IdentificadorLocal("185086693", "CADSUS-BULK", "2.16.840.1.113883.3.4594.3"),
						new IdentificadorLocal("0001753000", "CADSUS-UPDATE", "2.16.840.1.113883.3.4594.2"),
						new IdentificadorLocal("0001753000", "RES-BRASIL", "2.16.840.1.113883.3.4594")
				);

		assertThat(cidadao.getNacionalidade())
				.hasFieldOrPropertyWithValue(MBrasileiro.meta.codigoMunicipioNascimento.getAlias(), "330455")
				.hasFieldOrPropertyWithValue(MBrasileiro.meta.paisNascimento.getAlias(), Pais.BRASIL);

		assertThat(cidadao.getRg())
				.hasFieldOrPropertyWithValue(MRG.meta.dataEmissao.getAlias(), LocalDate.of(1989, 01, 27))
				.hasFieldOrPropertyWithValue(MRG.meta.numero.getAlias(), "01350389")
				.hasFieldOrPropertyWithValue(MRG.meta.orgaoEmissor.getAlias(), "81")
				.hasFieldOrPropertyWithValue(MRG.meta.uf.getAlias(), UF.RIO_DE_JANEIRO);
	}

}





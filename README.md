# contratoDAO
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DAO {

    // Estrutura para armazenar informações de uma proposta
    struct Proposta {
        string descricao;
        uint256 valor;
        address payable destinatario;
        uint256 votosFavor;
        uint256 votosContra;
        bool executada;
        mapping(address => bool) votou;
    }

    // Lista de todas as propostas
    Proposta[] public propostas;

    // Mapeia o saldo de cada membro da DAO
    mapping(address => uint256) public saldoMembros;

    // Total de fundos disponíveis na DAO
    uint256 public totalFundos;

    // Evento emitido quando uma nova proposta é criada
    event PropostaCriada(uint256 id, string descricao, uint256 valor, address destinatario);

    // Evento emitido quando uma proposta é votada
    event VotouProposta(uint256 id, address votante, bool favor);

    // Evento emitido quando uma proposta é executada
    event PropostaExecutada(uint256 id, bool sucesso);

    // Função para adicionar fundos à DAO
    function contribuir() public payable {
        saldoMembros[msg.sender] += msg.value;
        totalFundos += msg.value;
    }

    // Função para criar uma nova proposta
    function criarProposta(string memory _descricao, uint256 _valor, address payable _destinatario) public {
        require(saldoMembros[msg.sender] > 0, "Somente membros podem criar propostas");

        Proposta storage novaProposta = propostas.push();
        novaProposta.descricao = _descricao;
        novaProposta.valor = _valor;
        novaProposta.destinatario = _destinatario;
        novaProposta.executada = false;

        emit PropostaCriada(propostas.length - 1, _descricao, _valor, _destinatario);
    }

    // Função para votar em uma proposta
    function votarProposta(uint256 _id, bool _favor) public {
        require(_id < propostas.length, "Proposta inexistente");
        Proposta storage proposta = propostas[_id];

        require(saldoMembros[msg.sender] > 0, "Somente membros podem votar");
        require(!proposta.votou[msg.sender], "");
        require(!proposta.executada, "");

        if (_favor) {
            proposta.votosFavor += saldoMembros[msg.sender];
        } else {
            proposta.votosContra += saldoMembros[msg.sender];
        }

        proposta.votou[msg.sender] = true;

        emit VotouProposta(_id, msg.sender, _favor);
    }

    // Função para executar uma proposta se for aprovada
    function executarProposta(uint256 _id) public {
        require(_id < propostas.length, "Proposta inexistente");
        Proposta storage proposta = propostas[_id];

        require(!proposta.executada, "");
        require(proposta.votosFavor > proposta.votosContra, "");
        require(totalFundos >= proposta.valor, "");

        proposta.destinatario.transfer(proposta.valor);
        proposta.executada = true;
        totalFundos -= proposta.valor;

        emit PropostaExecutada(_id, true);
    }

    // Função para verificar o saldo total da DAO
    function verificarSaldoDAO() public view returns (uint256) {
        return totalFundos;
    }

    // Função para verificar o saldo de um membro específico
    function verificarSaldoMembro(address _membro) public view returns (uint256) {
        return saldoMembros[_membro];
    }
}

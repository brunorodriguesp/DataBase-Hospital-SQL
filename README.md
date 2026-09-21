# DataBase-Hospital-SQL
Simulação de um banco de dados de um hospital criado por mim em SQL.

-- =====================================================
-- BANCO DE DADOS: HOSPITAL
-- =====================================================

DROP DATABASE IF EXISTS hospital;
CREATE DATABASE hospital
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

USE hospital;

-- =====================================================
-- 1. TABELAS DE APOIO / CADASTROS BÁSICOS
-- =====================================================

CREATE TABLE departamentos (
    id_departamento     INT AUTO_INCREMENT PRIMARY KEY,
    nome                VARCHAR(100) NOT NULL,
    descricao           TEXT,
    andar               VARCHAR(20),
    telefone            VARCHAR(20),
    ativo               BOOLEAN DEFAULT TRUE,
    criado_em           DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

CREATE TABLE especialidades (
    id_especialidade    INT AUTO_INCREMENT PRIMARY KEY,
    nome                VARCHAR(100) NOT NULL UNIQUE,
    descricao           TEXT
) ENGINE=InnoDB;

CREATE TABLE planos_saude (
    id_plano            INT AUTO_INCREMENT PRIMARY KEY,
    nome                VARCHAR(100) NOT NULL,
    tipo                ENUM('Particular', 'Convenio', 'SUS') NOT NULL,
    cobertura           TEXT,
    ativo               BOOLEAN DEFAULT TRUE
) ENGINE=InnoDB;

CREATE TABLE medicamentos (
    id_medicamento      INT AUTO_INCREMENT PRIMARY KEY,
    nome                VARCHAR(150) NOT NULL,
    principio_ativo     VARCHAR(150),
    fabricante          VARCHAR(100),
    apresentacao        VARCHAR(100),          -- ex: comprimido 500mg
    controle_especial   BOOLEAN DEFAULT FALSE,
    estoque_atual       INT DEFAULT 0,
    estoque_minimo      INT DEFAULT 10,
    preco_custo         DECIMAL(10,2),
    ativo               BOOLEAN DEFAULT TRUE
) ENGINE=InnoDB;

-- =====================================================
-- 2. PESSOAS
-- =====================================================

CREATE TABLE pacientes (
    id_paciente         INT AUTO_INCREMENT PRIMARY KEY,
    nome                VARCHAR(150) NOT NULL,
    cpf                 VARCHAR(14) UNIQUE,
    rg                  VARCHAR(20),
    data_nascimento     DATE,
    sexo                ENUM('M','F','Outro') NOT NULL,
    tipo_sanguineo      ENUM('A+','A-','B+','B-','AB+','AB-','O+','O-'),
    estado_civil        VARCHAR(30),
    telefone            VARCHAR(20),
    celular             VARCHAR(20),
    email               VARCHAR(100),
    endereco            VARCHAR(200),
    bairro              VARCHAR(80),
    cidade              VARCHAR(80),
    estado              CHAR(2),
    cep                 VARCHAR(10),
    nome_mae            VARCHAR(150),
    contato_emergencia  VARCHAR(150),
    telefone_emergencia VARCHAR(20),
    id_plano            INT,
    numero_carteirinha  VARCHAR(50),
    observacoes         TEXT,
    ativo               BOOLEAN DEFAULT TRUE,
    criado_em           DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_plano) REFERENCES planos_saude(id_plano)
) ENGINE=InnoDB;

CREATE TABLE medicos (
    id_medico           INT AUTO_INCREMENT PRIMARY KEY,
    nome                VARCHAR(150) NOT NULL,
    crm                 VARCHAR(20) NOT NULL UNIQUE,
    cpf                 VARCHAR(14) UNIQUE,
    data_nascimento     DATE,
    sexo                ENUM('M','F','Outro'),
    telefone            VARCHAR(20),
    celular             VARCHAR(20),
    email               VARCHAR(100),
    id_especialidade    INT NOT NULL,
    id_departamento     INT,
    data_admissao       DATE,
    status              ENUM('Ativo','Licenca','Inativo') DEFAULT 'Ativo',
    criado_em           DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_especialidade) REFERENCES especialidades(id_especialidade),
    FOREIGN KEY (id_departamento) REFERENCES departamentos(id_departamento)
) ENGINE=InnoDB;

CREATE TABLE enfermeiros (
    id_enfermeiro       INT AUTO_INCREMENT PRIMARY KEY,
    nome                VARCHAR(150) NOT NULL,
    coren               VARCHAR(20) NOT NULL UNIQUE,
    cpf                 VARCHAR(14) UNIQUE,
    data_nascimento     DATE,
    sexo                ENUM('M','F','Outro'),
    telefone            VARCHAR(20),
    celular             VARCHAR(20),
    email               VARCHAR(100),
    id_departamento     INT,
    data_admissao       DATE,
    status              ENUM('Ativo','Licenca','Inativo') DEFAULT 'Ativo',
    criado_em           DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_departamento) REFERENCES departamentos(id_departamento)
) ENGINE=InnoDB;

-- =====================================================
-- 3. ESTRUTURA FÍSICA (QUARTOS E LEITOS)
-- =====================================================

CREATE TABLE quartos (
    id_quarto           INT AUTO_INCREMENT PRIMARY KEY,
    numero              VARCHAR(10) NOT NULL UNIQUE,
    tipo                ENUM('Enfermaria','Apartamento','UTI','Semi-UTI','Isolamento','Pediatria') NOT NULL,
    id_departamento     INT,
    capacidade          INT DEFAULT 1,
    status              ENUM('Disponivel','Ocupado','Manutencao','Higienizacao') DEFAULT 'Disponivel',
    valor_diaria        DECIMAL(10,2),
    FOREIGN KEY (id_departamento) REFERENCES departamentos(id_departamento)
) ENGINE=InnoDB;

CREATE TABLE leitos (
    id_leito            INT AUTO_INCREMENT PRIMARY KEY,
    id_quarto           INT NOT NULL,
    numero_leito        VARCHAR(10) NOT NULL,
    status              ENUM('Disponivel','Ocupado','Reservado','Manutencao') DEFAULT 'Disponivel',
    UNIQUE KEY uk_quarto_leito (id_quarto, numero_leito),
    FOREIGN KEY (id_quarto) REFERENCES quartos(id_quarto)
) ENGINE=InnoDB;

-- =====================================================
-- 4. ATENDIMENTO AMBULATORIAL (CONSULTAS)
-- =====================================================

CREATE TABLE consultas (
    id_consulta         INT AUTO_INCREMENT PRIMARY KEY,
    id_paciente         INT NOT NULL,
    id_medico           INT NOT NULL,
    data_hora           DATETIME NOT NULL,
    tipo                ENUM('Primeira Consulta','Retorno','Urgencia','Teleconsulta') DEFAULT 'Primeira Consulta',
    status              ENUM('Agendada','Confirmada','Em Andamento','Realizada','Cancelada','Falta') DEFAULT 'Agendada',
    motivo              TEXT,
    diagnostico         TEXT,
    observacoes         TEXT,
    valor               DECIMAL(10,2),
    criado_em           DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_paciente) REFERENCES pacientes(id_paciente),
    FOREIGN KEY (id_medico) REFERENCES medicos(id_medico),
    INDEX idx_consulta_data (data_hora),
    INDEX idx_consulta_paciente (id_paciente)
) ENGINE=InnoDB;

-- =====================================================
-- 5. INTERNAÇÕES
-- =====================================================

CREATE TABLE internacoes (
    id_internacao       INT AUTO_INCREMENT PRIMARY KEY,
    id_paciente         INT NOT NULL,
    id_medico_responsavel INT NOT NULL,
    id_leito            INT,
    data_entrada        DATETIME NOT NULL,
    data_saida_prevista DATETIME,
    data_saida_real     DATETIME,
    motivo              TEXT NOT NULL,
    diagnostico_entrada TEXT,
    diagnostico_saida   TEXT,
    status              ENUM('Internado','Alta','Transferido','Obito') DEFAULT 'Internado',
    tipo_internacao     ENUM('Clinica','Cirurgica','Obstetrica','Pediatrica','Psiquiatrica','UTI') NOT NULL,
    observacoes         TEXT,
    criado_em           DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_paciente) REFERENCES pacientes(id_paciente),
    FOREIGN KEY (id_medico_responsavel) REFERENCES medicos(id_medico),
    FOREIGN KEY (id_leito) REFERENCES leitos(id_leito)
) ENGINE=InnoDB;

-- =====================================================
-- 6. PRONTUÁRIO / HISTÓRICO
-- =====================================================

CREATE TABLE prontuarios (
    id_prontuario       INT AUTO_INCREMENT PRIMARY KEY,
    id_paciente         INT NOT NULL,
    id_medico           INT,
    id_consulta         INT,
    id_internacao       INT,
    data_registro       DATETIME DEFAULT CURRENT_TIMESTAMP,
    tipo_registro       ENUM('Anamnese','Evolucao','Prescricao','Procedimento','Alta','Outro') NOT NULL,
    descricao           TEXT NOT NULL,
    FOREIGN KEY (id_paciente) REFERENCES pacientes(id_paciente),
    FOREIGN KEY (id_medico) REFERENCES medicos(id_medico),
    FOREIGN KEY (id_consulta) REFERENCES consultas(id_consulta),
    FOREIGN KEY (id_internacao) REFERENCES internacoes(id_internacao)
) ENGINE=InnoDB;

-- =====================================================
-- 7. PRESCRIÇÕES
-- =====================================================

CREATE TABLE prescricoes (
    id_prescricao       INT AUTO_INCREMENT PRIMARY KEY,
    id_paciente         INT NOT NULL,
    id_medico           INT NOT NULL,
    id_internacao       INT,                   -- se for durante internação
    id_consulta         INT,                   -- se for ambulatorial
    data_prescricao     DATETIME DEFAULT CURRENT_TIMESTAMP,
    status              ENUM('Ativa','Suspensa','Concluida','Cancelada') DEFAULT 'Ativa',
    observacoes         TEXT,
    FOREIGN KEY (id_paciente) REFERENCES pacientes(id_paciente),
    FOREIGN KEY (id_medico) REFERENCES medicos(id_medico),
    FOREIGN KEY (id_internacao) REFERENCES internacoes(id_internacao),
    FOREIGN KEY (id_consulta) REFERENCES consultas(id_consulta)
) ENGINE=InnoDB;

CREATE TABLE itens_prescricao (
    id_item             INT AUTO_INCREMENT PRIMARY KEY,
    id_prescricao       INT NOT NULL,
    id_medicamento      INT NOT NULL,
    dosagem             VARCHAR(50) NOT NULL,  -- ex: 1 comprimido
    via_administracao   VARCHAR(50),           -- oral, EV, IM, SC...
    frequencia          VARCHAR(50),           -- 8/8h, 12/12h, SOS...
    duracao             VARCHAR(50),           -- 7 dias, contínuo...
    quantidade          INT,
    observacao          TEXT,
    FOREIGN KEY (id_prescricao) REFERENCES prescricoes(id_prescricao),
    FOREIGN KEY (id_medicamento) REFERENCES medicamentos(id_medicamento)
) ENGINE=InnoDB;

-- =====================================================
-- 8. EXAMES
-- =====================================================

CREATE TABLE exames (
    id_exame            INT AUTO_INCREMENT PRIMARY KEY,
    nome                VARCHAR(150) NOT NULL,
    tipo                ENUM('Laboratorial','Imagem','Funcional','Outro') NOT NULL,
    descricao           TEXT,
    preparo             TEXT,
    valor               DECIMAL(10,2)
) ENGINE=InnoDB;

CREATE TABLE solicitacoes_exame (
    id_solicitacao      INT AUTO_INCREMENT PRIMARY KEY,
    id_paciente         INT NOT NULL,
    id_medico           INT NOT NULL,
    id_exame            INT NOT NULL,
    id_consulta         INT,
    id_internacao       INT,
    data_solicitacao    DATETIME DEFAULT CURRENT_TIMESTAMP,
    data_realizacao     DATETIME,
    status              ENUM('Solicitado','Coletado','Em Analise','Concluido','Cancelado') DEFAULT 'Solicitado',
    resultado           TEXT,
    laudo               TEXT,
    observacoes         TEXT,
    FOREIGN KEY (id_paciente) REFERENCES pacientes(id_paciente),
    FOREIGN KEY (id_medico) REFERENCES medicos(id_medico),
    FOREIGN KEY (id_exame) REFERENCES exames(id_exame),
    FOREIGN KEY (id_consulta) REFERENCES consultas(id_consulta),
    FOREIGN KEY (id_internacao) REFERENCES internacoes(id_internacao)
) ENGINE=InnoDB;

-- =====================================================
-- 9. CIRURGIAS
-- =====================================================

CREATE TABLE salas_cirurgicas (
    id_sala             INT AUTO_INCREMENT PRIMARY KEY,
    nome                VARCHAR(50) NOT NULL UNIQUE,
    tipo                ENUM('Geral','Ortopedica','Cardiaca','Neurologica','Obstetrica') NOT NULL,
    status              ENUM('Disponivel','Ocupada','Manutencao') DEFAULT 'Disponivel'
) ENGINE=InnoDB;

CREATE TABLE cirurgias (
    id_cirurgia         INT AUTO_INCREMENT PRIMARY KEY,
    id_paciente         INT NOT NULL,
    id_medico_cirurgiao INT NOT NULL,
    id_sala             INT,
    id_internacao       INT,
    data_hora_inicio    DATETIME NOT NULL,
    data_hora_fim       DATETIME,
    tipo_cirurgia       VARCHAR(150) NOT NULL,
    descricao           TEXT,
    status              ENUM('Agendada','Em Andamento','Concluida','Cancelada') DEFAULT 'Agendada',
    anestesia           VARCHAR(100),
    observacoes         TEXT,
    FOREIGN KEY (id_paciente) REFERENCES pacientes(id_paciente),
    FOREIGN KEY (id_medico_cirurgiao) REFERENCES medicos(id_medico),
    FOREIGN KEY (id_sala) REFERENCES salas_cirurgicas(id_sala),
    FOREIGN KEY (id_internacao) REFERENCES internacoes(id_internacao)
) ENGINE=InnoDB;

-- =====================================================
-- 10. FATURAMENTO (SIMPLIFICADO)
-- =====================================================

CREATE TABLE faturas (
    id_fatura           INT AUTO_INCREMENT PRIMARY KEY,
    id_paciente         INT NOT NULL,
    id_internacao       INT,
    id_consulta         INT,
    data_emissao        DATE NOT NULL,
    valor_total         DECIMAL(12,2) NOT NULL,
    valor_pago          DECIMAL(12,2) DEFAULT 0,
    status              ENUM('Aberta','Paga','Parcialmente Paga','Cancelada') DEFAULT 'Aberta',
    forma_pagamento     ENUM('Dinheiro','Cartao','PIX','Convenio','SUS'),
    observacoes         TEXT,
    FOREIGN KEY (id_paciente) REFERENCES pacientes(id_paciente),
    FOREIGN KEY (id_internacao) REFERENCES internacoes(id_internacao),
    FOREIGN KEY (id_consulta) REFERENCES consultas(id_consulta)
) ENGINE=InnoDB;

-- =====================================================
-- DADOS DE EXEMPLO (OPCIONAL - pode comentar se quiser)
-- =====================================================

INSERT INTO departamentos (nome, descricao, andar) VALUES
('Clínica Médica', 'Atendimento clínico geral', '2º Andar'),
('Cirurgia', 'Centro cirúrgico e pós-operatório', '3º Andar'),
('Pediatria', 'Atendimento infantil', '1º Andar'),
('UTI', 'Unidade de Terapia Intensiva', '4º Andar'),
('Emergência', 'Pronto-socorro 24h', 'Térreo');

INSERT INTO especialidades (nome) VALUES
('Clínica Geral'), ('Cardiologia'), ('Ortopedia'),
('Pediatria'), ('Ginecologia e Obstetrícia'),
('Neurologia'), ('Cirurgia Geral'), ('Anestesiologia');

INSERT INTO planos_saude (nome, tipo) VALUES
('Particular', 'Particular'),
('Unimed', 'Convenio'),
('Bradesco Saúde', 'Convenio'),
('SUS', 'SUS');

INSERT INTO medicamentos (nome, principio_ativo, apresentacao, estoque_atual) VALUES
('Dipirona 500mg', 'Dipirona sódica', 'Comprimido', 500),
('Amoxicilina 500mg', 'Amoxicilina', 'Cápsula', 300),
('Losartana 50mg', 'Losartana potássica', 'Comprimido', 400),
('Soro Fisiológico 0,9%', 'Cloreto de sódio', 'Bolsa 500ml', 200),
('Paracetamol 750mg', 'Paracetamol', 'Comprimido', 600);

INSERT INTO medicos (nome, crm, id_especialidade, id_departamento, status) VALUES
('Dr. Carlos Mendes', 'CRM-12345', 1, 1, 'Ativo'),
('Dra. Ana Paula Souza', 'CRM-23456', 2, 1, 'Ativo'),
('Dr. Roberto Lima', 'CRM-34567', 7, 2, 'Ativo'),
('Dra. Fernanda Costa', 'CRM-45678', 4, 3, 'Ativo');

INSERT INTO pacientes (nome, cpf, data_nascimento, sexo, tipo_sanguineo, telefone, id_plano) VALUES
('João da Silva', '123.456.789-00', '1985-03-15', 'M', 'O+', '(11) 99999-1111', 2),
('Maria Oliveira', '987.654.321-00', '1990-07-22', 'F', 'A+', '(11) 98888-2222', 3),
('Pedro Santos', '456.789.123-00', '1978-11-05', 'M', 'B-', '(11) 97777-3333', 4);

-- =====================================================
-- ÍNDICES ADICIONAIS PARA PERFORMANCE
-- =====================================================

CREATE INDEX idx_paciente_nome ON pacientes(nome);
CREATE INDEX idx_medico_crm ON medicos(crm);
CREATE INDEX idx_internacao_status ON internacoes(status);
CREATE INDEX idx_consulta_status ON consultas(status);

-- =====================================================
-- FIM DO SCRIPT
-- =====================================================
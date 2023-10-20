# Processando-e-analisando-em-Power-BI_Empresa-fictcia_Azure-Company
O projeto é o resultado dos estudo Bootcamp Santander 2023 Data Science Python. Trata-se de uma visão analítica dos dados de uma empresa(fictícia), hospedada no banco de dados MySQL do Azure e visualizados usando PowerBI.

## Acesso ao Projeto:
[PowerBI Desktop](https://github.com/Vilson1984/Processando-e-analisando-em-Power-BI_Empresa-fictcia_Azure-Company/blob/master/Desafio%204%20-%20FINAL%20-%20Processando%20e%20Transformando%20dados%20com%20Power%20BI.pbix)

[PowerBi Service - Online](https://app.powerbi.com/links/s9oeN5x3cY?ctid=e303a7fd-6ce9-4d1d-a70c-fc9642bbcfeb&pbi_source=linkShare)
        
    

#### Objetivo:

    Criar banco de dados.
    Extrair dados.
    Transformar os dados, no Power Query, para fins analíticos.
    Visualizar os dados transformados usando PowerBI.


#### Tecnologias Utilizadas:

    Azure: Microsoft Azure é uma plataforma de computação em nuvem que oferece uma variedade de serviços como poder de computação, opções de armazenamento e recursos de rede. Ele permite que as empresas executem seus aplicativos, armazenem dados e realizem análises em um ambiente seguro e escalável.

    MySQL: Um sistema de gerenciamento de banco de dados relacional de código aberto amplamente utilizado que não é exclusivo da Microsoft. É conhecido por sua velocidade, confiabilidade e facilidade de uso. Embora seja frequentemente usado em conjunto com o Azure, não é fornecido pela Microsoft no site do Power BI.

    PowerBI Desktop: uma poderosa ferramenta de análise de negócios da Microsoft que permite visualizar seus dados e compartilhar insights em toda a sua organização ou incorporá-los em um aplicativo ou site. Ele oferece uma interface baseada em desktop para a criação de relatórios e painéis.


#### Diretrizes para transformação dos dados:

    1. Verifique os cabeçalhos e tipos de dados
    2. Modifique os valores monetários para o tipo double preciso
    3. Verifique a existência dos nulos e analise a remoção
    4. Os employees com nulos em Super_ssn podem ser os gerentes. Verifique se há algum colaborador sem gerente
    5. Verifique se há algum departamento sem gerente
    6. Se houver departamento sem gerente, suponha que você possui os dados e preencha as lacunas
    7. Verifique o número de horas dos projetos
    8. Separar colunas complexas
    9. Mesclar consultas employee e departament para criar uma tabela employee com o nome dos departamentos associados aos colaboradores. A mescla terá como base a tabela employee. Fique atento, essa informação influencia no tipo de junção
    10. Neste processo elimine as colunas desnecessárias.
    11. Realize a junção dos colaboradores e respectivos nomes dos gerentes . Isso pode ser feito com consulta SQL ou pela mescla de tabelas com Power BI. Caso utilize SQL, especifique no README a query utilizada no processo.
    12. Mescle as colunas de Nome e Sobrenome para ter apenas uma coluna definindo os nomes dos colaboradores
    13. Mescle os nomes de departamentos e localização. Isso fará que cada combinação departamento-local seja único. Isso irá auxiliar na criação do modelo estrela em um módulo futuro.
    14. Explique por que, neste caso supracitado, podemos apenas utilizar o mesclar e não o atribuir.
    15. Agrupe os dados a fim de saber quantos colaboradores existem por gerente
    16. Elimine as colunas desnecessárias, que não serão usadas no relatório, de cada tabela


#### Método:

    --Criando base de dados Azure_company
    create schema if not exists azure_company;

    --Usando o BD
    use azure_company;

  
    select * from information_schema.table_constraints

    where constraint_schema = 'azure_company';

    --Criando tabela employee
    CREATE TABLE employee(

    Fname varchar(15) not null,

      Minit char,

      Lname varchar(15) not null,

      Ssn char(9) not null, 

      Bdate date,

      Address varchar(30),

      Sex char,

      Salary decimal(10,2),

      Super_ssn char(9),

      Dno int not null,

      constraint chk_salary_employee check (Salary> 2000.0),

      constraint pk_employee primary key (Ssn)

    );

    --Alterando valor da tabela employee
    alter table employee modify Dno int not null default 1;

    --Criando tabela Departamento
    create table departament(

    Dname varchar(15) not null,

      Dnumber int not null,

      Mgr_ssn char(9) not null,

      Mgr_start_date date, 

      Dept_create_date date,

      constraint chk_date_dept check (Dept_create_date < Mgr_start_date),

      constraint pk_dept primary key (Dnumber),

      constraint unique_name_dept unique(Dname),

      foreign key (Mgr_ssn) references employee(Ssn)

    );

    --Adicionando chave estrangeira
    alter table departament add constraint fk_dept foreign key(Mgr_ssn) references employee(Ssn) on update cascade;

    --Criando tabela Departamentos Locais
    create table dept_locations(

    Dnumber int not null,

    Dlocation varchar(15) not null,

      constraint pk_dept_locations primary key (Dnumber, Dlocation)

    );

    --Adicionando chave estrangeira
    alter table dept_locations 

    add constraint fk_dept_locations foreign key (Dnumber) references departament(Dnumber)

    on delete cascade

      on update cascade;

    --Criando tabela Projeto
    create table project(

    Pname varchar(15) not null,

    Pnumber int not null,

      Plocation varchar(15),

      Dnum int not null,

      primary key (Pnumber),

      constraint unique_project unique (Pname),

      constraint fk_project foreign key (Dnum) references departament(Dnumber)

    );


    --Criando tabela Works on
    create table works_on(

    Essn char(9) not null,

      Pno int not null,

      Hours decimal(3,1) not null,

      primary key (Essn, Pno),

      constraint fk_employee_works_on foreign key (Essn) references employee(Ssn),

      constraint fk_project_works_on foreign key (Pno) references project(Pnumber)

    );

    --Criando tabela dependentes
    create table dependent(

    Essn char(9) not null,

      Dependent_name varchar(15) not null,

      Sex char,

      Bdate date,

      Relationship varchar(8),

      primary key (Essn, Dependent_name),

      constraint fk_dependent foreign key (Essn) references employee(Ssn)

    );

    --Inserindo registros na tabela employee
    insert into employee values ('John', 'B', 'Smith', 123456789, '1965-01-09', '731-Fondren-Houston-TX', 'M', 30000, 333445555, 5), ('Franklin', 'T', 'Wong', 333445555, '1955-12-08', '638-Voss-Houston-TX', 'M', 40000, 888665555, 5), ('Alicia', 'J', 'Zelaya', 999887777, '1968-01-19', '3321-Castle-Spring-TX', 'F', 25000, 987654321, 4), ('Jennifer', 'S', 'Wallace', 987654321, '1941-06-20', '291-Berry-Bellaire-TX', 'F', 43000, 888665555, 4), ('Ramesh', 'K', 'Narayan', 666884444, '1962-09-15', '975-Fire-Oak-Humble-TX', 'M', 38000, 333445555, 5), ('Joyce', 'A', 'English', 453453453, '1972-07-31', '5631-Rice-Houston-TX', 'F', 25000, 333445555, 5), ('Ahmad', 'V', 'Jabbar', 987987987, '1969-03-29', '980-Dallas-Houston-TX', 'M', 25000, 987654321, 4), ('James', 'E', 'Borg', 888665555, '1937-11-10', '450-Stone-Houston-TX', 'M', 55000, NULL, 1);

    --Inserindo registros na tabela dependentes
    insert into dependent values (333445555, 'Alice', 'F', '1986-04-05', 'Daughter'), (333445555, 'Theodore', 'M', '1983-10-25', 'Son'), (333445555, 'Joy', 'F', '1958-05-03', 'Spouse'), (987654321, 'Abner', 'M', '1942-02-28', 'Spouse'), (123456789, 'Michael', 'M', '1988-01-04', 'Son'), (123456789, 'Alice', 'F', '1988-12-30', 'Daughter'), (123456789, 'Elizabeth', 'F', '1967-05-05', 'Spouse');

    --Inserindo registros na tabela departamentos
    insert into departament values ('Research', 5, 333445555, '1988-05-22','1986-05-22'), ('Administration', 4, 987654321, '1995-01-01','1994-01-01'), ('Headquarters', 1, 888665555,'1981-06-19','1980-06-19');

    --Inserindo registros na tabela departamentos locais
    insert into dept_locations values (1, 'Houston'), (4, 'Stafford'), (5, 'Bellaire'), (5, 'Sugarland'), (5, 'Houston');

    --Inserindo registros na tabela Projetos
    insert into project values ('ProductX', 1, 'Bellaire', 5), ('ProductY', 2, 'Sugarland', 5), ('ProductZ', 3, 'Houston', 5), ('Computerization', 10, 'Stafford', 4), ('Reorganization', 20, 'Houston', 1), ('Newbenefits', 30, 'Stafford', 4);

    --Inserindo registros na tabela works_on
    insert into works_on values (123456789, 1, 32.5), (123456789, 2, 7.5), (666884444, 3, 40.0), (453453453, 1, 20.0), (453453453, 2, 20.0), (333445555, 2, 10.0), (333445555, 3, 10.0), (333445555, 10, 10.0), (333445555, 20, 10.0), (999887777, 30, 30.0), (999887777, 10, 10.0), (987987987, 10, 35.0), (987987987, 30, 5.0), (987654321, 30, 20.0), (987654321, 20, 15.0), (888665555, 20, 0.0);




TOC TOC
- Olá, teria tempo pra aceitar ~~Jesus~~ [AutoMapper](http://automapper.org/ "AutoMapper") em sua vida?


Sim, amigos bípedes. O evangelista de AutoMapper tentando convencer vocês a usar esta maravilha.
Depois de uma conversa calorosa no Facebook explicando como funcionava pra um amigo, resolvi replicar pra cá parte da conversa editando pra ficar compacto e mais claro.
Perdão se falhei em minha missão (ficou maior que eu esperava).
Desconsiderem os erros de português ou algum erro de sintaxe porque escrevi direto no Facebook.

Por que usar AutoMapper?
Simples, porque eu uso e vou usar em todos os projetos que eu trabalhar MUHUAHUA
Cedo ou tarde vocês terão que dar manutenção neles então estejam preparados.
Mentira, galera. É só pra vocês conhecerem uma outra abordagem do nosso modo de trabalho.
Pretendo escrever mais alguns. Espero que vocês também tragam mais tutoriais pra cá. 

Então vamos começar, mas antes... Vamos voltar um pouco.
No princípio criou Deus o céu e a terra...
Não, pera, voltei demais.

## ViewModel

Galerinha, acredito que todos vocês já façam uso de ViewModel então vou ser breve.

Muito basicamente uma ViewModel seria a representação dos dados que você gostaria de exibir em uma view.
Ela não tem ligação restrita com Asp.Net MVC. É um conceito que pode ser aplicado a varias outras arquiteturas.

Mas vamos focar no Asp.Net MVC. Vamos supor que você queira exibir em uma View a edição de um Perfil e de um Departamento ao mesmo tempo.
Como vocês sabem cada View aceita apenas 1 objeto associado a mesma.

Como resolver nesse caso? Nada de ViewBag, ViewData por favor. ViewBag/ViewData só pra campos externos (filtros, listas etc)

Então como proceder?

Cria-se um "container" que agregue os dois objetos

    public class FormViewModel 
	{
	    public Profile Perfil { get; set; } 
	    public Department Departamento { get; set; } 
    }

Agora você consegue passar para a sua View os dois campos em um objeto só. 
Idiota de tão simples, não?

Mas nem só pra "Containers" o uso de ViewModel é recomendado. Basicamente, é sempre interessante usar uma ViewModel em suas views ao invés do Model direto do banco.
Por que? Porque você agrega a sua View um comportamento real, sem nada desnecessário.
É praticamente uma documentação do que a View necessita.

Isso deixa mais claro a sua intenção quando alguém for dar manutenção em seu código.

Outro bom motivo é não ~~cagar~~ bagunçar seu modelo com *annotations* de validação/exibição:

    public class Profile {
    
    	public int Id { get; set; } 
    
    	[Required]
    	[Display(Name = "Nome Completo:")]	
    	public string Nome { get; set; }
    
    	[Required] 
    	[EmailAddress]	
    	[Display(Name = "E-mail:")]
    	[StringLength(100, ErrorMessage = "A {0} deve ter pelo menos {2} caracteres.", MinimumLength = 6)]
    	public string Email { get; set; }
    }

Ainda temos a questão de segurança. Você controla exatamente o que deve ser exibido e principalmente o que deve ser postado de volta.

Imagine que a classe Profile seja a seguinte:

    public class ProfileViewModel {
	    public int Id { get; set; } 
	    public string Nome { get; set; } 
	    public string Foto { get; set; }
	    public float Salario { get; set; }
    }

Se, por ventura, você passar este model a um formulario de edição (que só permite editar o nome), o usuário pode forçar um POST com o parâmetro salário alterado.
A alteração não esperada por você irá para o banco caso não haja algum bloqueio da sua parte.

Enfim, existem N motivos pra utilizar o pattern ViewModel.
Não vou citar todos. Apenas mais um: é uma boa pratica adotada por grandes empresas e bons programadores.
A menos que possua bons argumentos pra não usar (preguiça não vale), use sempre.

Bom, sabendo disso agora temos com o que trabalhar.

Veja o código abaixo:

    public ActionResult GetDepartament(int id)
    {
	    Department department = db.Departments.Find(id);
	    
	    if(department == null)
	    {
	    	return HttpNotFound();
	    }
	    
	    DepartmentViewModel departmentViewModel = new DepartmentViewModel();
	    
	    departmentViewModel.Id = department.Id;
	    departmentViewModel.Name = department.Name;
	    departmentViewModel.Area = department.Area;
	    
	    return View(departmentViewModel);
    }

Basicamente seria assim uma maneira aceitável de trabalhar.
Você procura no banco a entidade que você quer e relaciona com a sua ViewModel apenas os campos que você deseja.

Esse exemplo é tranquilo. São apenas três campos.  
Mas imagina ter que fazer essa associação de igual pra igual pra varios campos por varias vezes?

Você vai acabar achando mais vantajoso passar o Model direto. Mas não caia em tentação pois é ai que entra o [AutoMapper](http://automapper.org/ "AutoMapper")


## AutoMapper ##

O [AutoMapper](http://automapper.org/ "AutoMapper") é uma library de mapeamento entre objetos. Ele poupa o chato trabalho de fazer associação entre campos.
Ele não é a única ferramenta a fazer isso mas é de longe a mais famosa.

Como ele funciona?

Ele associa uma propriedade a outra através do nome das mesmas. 
Por exemplo, se você possui no seu Model uma propriedade 'Name' ele irá buscar na sua ViewModel um campo para relacionar que também se chama 'Name'.
Pronto, a AutoMagic acontece.

Como utilizar?

Primeiro você deve criar o mapeamento entre as classes que você deseja.
No nosso caso:

    
    Mapper.CreateMap<Department, DepartmentViewModel>();

> Atenção: O AutoMapper, em suas versões mais recentes, mudou a forma de criação dos maps mas acredito que as mesmas ainda aceitam esse modo de mapear.

No código acima estou dizendo que Department será mapeado para DepartmentViewModel.

E pra fazer o mapeamento eu utilizo:

    DepartmentViewModel departmentViewModel = Mapper.Map<DepartmentViewModel>(department);

Onde, entre as tags eu passo o tipo do objeto de destino e, entre parenteses eu passo o objeto que eu quero copiar para o destino.

Ficaria algo assim:

   	public ActionResult GetDepartament(int id)
    {
    	Department department = db.Departments.Find(id);
    	
    	if(department == null)
    	{
    		return HttpNotFound();
    	}
    	   
    	//Mais facil hein??
    	DepartmentViewModel departmentViewModel = Mapper.Map<DepartmentViewModel>(department);
    	
    	return View(departmentViewModel);
    }

O processo inverso (associar uma viewModel a um model):

    public ActionResult Edit(int id, DepartmentViewModel viewModel)
    {
		if(ModelState.IsValid)
		{
			using(var db = new AppContext())
			{
				Department department = db.Departments.Find(id);
				
				if(department == null)
				{
					return HttpNotFound();
				}
		   	
				Mapper.Map(viewModel, department);

				db.SaveChanges();

				return RedirectToAction("Index");
			}			
    	}
    
    	return View(departmentViewModel);
    }

Bem legal.

Ainda podemos melhorar um pouco as coisas.  
Basicamente, se você não especificar quais colunas você deseja trazer do banco(fazer uma projeção), o EntityFramework trará todas.

Veja o exemplo. No LINQ abaixo eu não especifico quais colunas eu vou utilizar:


    var departments = db.Departments.Where(q => q.Name == "Departamento 1").First();

Logo, a query gerada será:

    SELECT 
        [Extent1].[DepartmentID] AS [DepartmentID], 
        [Extent1].[Name] AS [Name], 
        [Extent1].[Budget] AS [Budget], 
        [Extent1].[StartDate] AS [StartDate], 
        [Extent1].[InstructorID] AS [InstructorID], 
        [Extent1].[RowVersion] AS [RowVersion]
    FROM [dbo].[Department] AS [Extent1]
    WHERE N'Departamento 1' = [Extent1].[Name]


Imagina que você precisa de somente 2 campos na sua ViewModel.
Desnecessariamente você trará tudo.

Você pode fazer o mapeamento Model > ViewModel na mão:

    var departments = db.Departments.Where(q => q.Name == "Departamento 1").Select(d => new DepartmentViewModel()
    {
	    DepartmentID = d.DepartmentID,
	    Name = d.Name
    }).First();
    
O que geraria a query:

     SELECT 
	    [Extent1].[DepartmentID] AS [DepartmentID], 
	    [Extent1].[Name] AS [Name]
    FROM [dbo].[Department] AS [Extent1]
	WHERE N'Departamento 1' = [Extent1].[Name]

É um bloco de código bem chato e repetitido.  
Para isso, o AutoMapper provê uma solução bem interessante: [Queryable Extensions (QE)](https://github.com/AutoMapper/AutoMapper/wiki/Queryable-Extensions "Queryable Extensions (QE)")

A esse namespace foi adicionada a função Project().To<>() que faz justamente o que fazemos no bloco de código acima só que em uma motherfucker linha:

    var departmentViewModel = db.Departments.Project().To<DepartmentViewModel>().First(d => d.Name == "Departamento 1");


> Só uma observação. Não sei se todos sabem, mas o Entity só vai ao banco quando ele, de fato, julgar necessário trazer os objetos.
> E quando isso acontece? Pode ser um ToList(), ToArray(), First() etc.
> Enquanto for apenas uma IQueryable, ele está apenas montando a query.

Então, é isso. Se alguem tiver alguma duvida, objeção ou uma ferramenta melhor me procure no SPARK. Estou aberto a discussões.
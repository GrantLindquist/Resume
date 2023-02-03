# Overview

MonkeyBusiness is full-stack web application using Spring Boot and Thymeleaf connected to a SQL database. “MonkeyBusiness” is a simple, mock e-commerce website that adheres to basic security principles such as user authentication and password encryption.

![MonkeyBusiness homepage](https://user-images.githubusercontent.com/78822631/216498176-3945bc0f-82a9-4b0a-9230-f1cca857319b.jpg)

---

# Design Documentation

### API Design
![CST339milestone7p1](https://user-images.githubusercontent.com/78822631/216498573-3333e459-da33-44b7-8ef0-08263beb131f.PNG)

### Sitemap
![CST339milestone7p2](https://user-images.githubusercontent.com/78822631/216498590-b559d98c-45b9-46f0-887b-27df19498a1c.PNG)

---

# Code Snippets

### MVC & N-Layer architecture
This code displays the N-layer architecture of this project. You can see the Product objects being extracted from the database within the Data layer, and then being passed through the Business layer in order for the Controller to display the objects in the Presentation layer.

*ProductsDataService.java*
```
public List<ProductModel> findAll()
{
	String sql = "SELECT * FROM monkeybusiness.product";
	List<ProductModel> products = new ArrayList<ProductModel>();
	try 
	{
		// Executes SQL query
		SqlRowSet srs = jdbcTemplateObject.queryForRowSet(sql);
		// Loops through results of query
		while(srs.next()) 
		{
			products.add(new ProductModel(srs.getString("name"), 
								srs.getString("description"),
								srs.getString("size"),
								srs.getDouble("price"),
								srs.getInt("productId")));
		}
	}
	catch(Exception e) 
	{
		e.printStackTrace();
	}
	
	return products;
}
```
*ProductsBusinessService.java*
```
/**
* Returns all products from database
* @return list of products
*/
public List<ProductModel> getProducts()
{
	return service.findAll();
}
```
*MenuController.java*
```
/**
* Displays main menu.
* @param model
* @return login html file
*/
@GetMapping("/")
public String display(Model model) 
{
	// If user is logged in, display products
	model.addAttribute("products", productBusinessService.getProducts());
	
	return "index";
}
```

### Spring Security configuration
This piece of code activates Spring Security for my project. Additionally, specifies which pages can be accessed by an unauthorized user. Every other page requires the user to sign into an account before accessing. This code also preforms basic encryption/decryption on the user's password before it is entered or extratced from the database.
```
/**
 * This class allows for the application to use Spring Security.
 */
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter
{
	// User service that handles authroization
	@Autowired
	UserBusinessService service;
	
	// Encodes passwords
	@Lazy
	@Autowired
	PasswordEncoder passwordEncoder;
	
	// Replaces NOOP password encoder with bcrypt
	@Bean(name = "passwordEncoder")
	BCryptPasswordEncoder passwordEncoder()
	{
		return new BCryptPasswordEncoder();
	}
	
	/**
	 * Confugures spring security.
	 */
	@Override
	protected void configure(HttpSecurity http) throws Exception 
	{	
		http.csrf().disable()
		.authorizeRequests()
		// Anyone may access these pages:
	        .antMatchers("/*", "/login/*", "/register/*", "/js/*", "/css/*", "/images/*").permitAll()
	        .anyRequest().authenticated()
	        .and()
		.formLogin()
			.loginPage("/login")
			// Login submitted with input id & name of "username" and "password" is treated
			// as official authentication
			.usernameParameter("username")
			.passwordParameter("password")
			.permitAll()
			.defaultSuccessUrl("/", true)
			.and()
			// Undos authentication
		.logout()
			.logoutUrl("/logout")
			.invalidateHttpSession(true)
			.clearAuthentication(true)
			.permitAll()
			.logoutSuccessUrl("/");
	}
	
	/**
	 * Autwired configure which takes care of the bean creation automatically.
	 */
	@Autowired
	public void configure(AuthenticationManagerBuilder auth) throws Exception 
	{
		// Handles user authorization
		auth.userDetailsService(service).passwordEncoder(passwordEncoder);
	}
}
```

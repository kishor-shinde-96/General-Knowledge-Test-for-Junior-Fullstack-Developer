# General Knowledge Test for Junior Fullstack Developer

## Section 1: Symfony

1. **Configuration Question:**
- Describe the basic steps to set up a project in Symfony.
  - Install Symfony CLI and run `symfony new project_name --full` to create a new project.
  - Configure the database in `.env` file.
  - Run `php bin/console doctrine:database:create` to create the database.
  - Use `php bin/console make:migration` and `php bin/console doctrine:migrations:migrate` to set up database schema.
  - Start the development server with `symfony serve`.

2. **Code Question:**
- Create a controller in Symfony that handles a route `/hello/{name}` and returns a personalized greeting. If the name is not provided, it should return a default greeting "Hello World". Optionally, implement a unit test to verify the route returns the correct greeting.

  ```php
  // src/Controller/GreetingController.php
  namespace App\Controller;

  use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
  use Symfony\Component\HttpFoundation\Response;
  use Symfony\Component\Routing\Annotation\Route;

  class GreetingController extends AbstractController
  {
      /**
       * @Route("/hello/{name}", defaults={"name"="World"}, name="greeting")
       */
      public function greet(string $name): Response
      {
          return new Response(sprintf('Hello %s', $name));
      }
  }

  // tests/Controller/GreetingControllerTest.php
  namespace App\Tests\Controller;

  use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

  class GreetingControllerTest extends WebTestCase
  {
      public function testGreetWithName()
      {
          $client = static::createClient();
          $crawler = $client->request('GET', '/hello/John');
          $this->assertResponseIsSuccessful();
          $this->assertSelectorTextContains('body', 'Hello John');
      }

      public function testGreetWithoutName()
      {
          $client = static::createClient();
          $crawler = $client->request('GET', '/hello');
          $this->assertResponseIsSuccessful();
          $this->assertSelectorTextContains('body', 'Hello World');
      }
  }
  ```

3. **Theoretical Question:**
- Explain the architecture of Symfony and how a typical project is organized in terms of folders and files.
  - Symfony follows an MVC (Model-View-Controller) architecture.
  - Common folders include:
    - `src/`: Contains application source code.
    - `config/`: Configuration files for services, routes, etc.
    - `templates/`: Twig templates for views.
    - `public/`: Entry point for the application (index.php).
    - `var/`: Cache and logs.
    - `vendor/`: Third-party dependencies.
    - `tests/`: Test files.

4. **Code Question:**
- Write a service in Symfony that is injected into a controller and performs a basic mathematical operation (e.g., adding two numbers). What configurations are necessary to use it? Optionally, implement a unit test to verify the service's correct functionality.

  ```php
  // src/Service/CalculatorService.php
  namespace App\Service;

  class CalculatorService
  {
      public function add(int $a, int $b): int
      {
          return $a + $b;
      }
  }

  // src/Controller/CalculatorController.php
  namespace App\Controller;

  use App\Service\CalculatorService;
  use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
  use Symfony\Component\HttpFoundation\Response;
  use Symfony\Component\Routing\Annotation\Route;

  class CalculatorController extends AbstractController
  {
      private $calculator;

      public function __construct(CalculatorService $calculator)
      {
          $this->calculator = $calculator;
      }

      /**
       * @Route("/add/{a}/{b}", name="add")
       */
      public function add(int $a, int $b): Response
      {
          $result = $this->calculator->add($a, $b);
          return new Response(sprintf('Result: %d', $result));
      }
  }

  // services.yaml (Configuration)
  services:
      App\Service\CalculatorService: ~
  ```

5. **Code Question:**
- Show how to create a form in Symfony for a User entity with fields `username` and `email`.

  ```php
  // src/Entity/User.php
  namespace App\Entity;

  use Doctrine\ORM\Mapping as ORM;

  /**
   * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
   */
  class User
  {
      /**
       * @ORM\Id
       * @ORM\GeneratedValue
       * @ORM\Column(type="integer")
       */
      private $id;

      /**
       * @ORM\Column(type="string", length=100)
       */
      private $username;

      /**
       * @ORM\Column(type="string", length=100)
       */
      private $email;

      // getters and setters
  }

  // src/Form/UserType.php
  namespace App\Form;

  use App\Entity\User;
  use Symfony\Component\Form\AbstractType;
  use Symfony\Component\Form\FormBuilderInterface;
  use Symfony\Component\OptionsResolver\OptionsResolver;
  use Symfony\Component\Form\Extension\Core\Type\EmailType;
  use Symfony\Component\Form\Extension\Core\Type\TextType;

  class UserType extends AbstractType
  {
      public function buildForm(FormBuilderInterface $builder, array $options)
      {
          $builder
              ->add('username', TextType::class)
              ->add('email', EmailType::class);
      }

      public function configureOptions(OptionsResolver $resolver)
      {
          $resolver->setDefaults([
              'data_class' => User::class,
          ]);
      }
  }

  // src/Controller/UserController.php
  namespace App\Controller;

  use App\Entity\User;
  use App\Form\UserType;
  use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
  use Symfony\Component\HttpFoundation\Request;
  use Symfony\Component\HttpFoundation\Response;
  use Symfony\Component\Routing\Annotation\Route;

  class UserController extends AbstractController
  {
      /**
       * @Route("/user/new", name="user_new")
       */
      public function new(Request $request): Response
      {
          $user = new User();
          $form = $this->createForm(UserType::class, $user);

          $form->handleRequest($request);

          if ($form->isSubmitted() && $form->isValid()) {
              $entityManager = $this->getDoctrine()->getManager();
              $entityManager->persist($user);
              $entityManager->flush();

              return $this->redirectToRoute('user_success');
          }

          return $this->render('user/new.html.twig', [
              'form' => $form->createView(),
          ]);
      }
  }

  // templates/user/new.html.twig
  {{ form_start(form) }}
  {{ form_widget(form) }}
  <button type="submit">Submit</button>
  {{ form_end(form) }}
  ```

6. **Theoretical Question:**
- Explain the concept of Dependency Injection (DI) in Symfony and how it is configured.
  - Dependency Injection is a design pattern where an object receives other objects it depends on.
  - In Symfony, services are registered in the service container, and dependencies are injected via constructors or setters.
  - Configuration is done in `services.yaml` or using annotations.

7. **Code Question:**
- Write a Doctrine query in Symfony to fetch all records from a `Product` entity where the price is greater than 100.

  ```php
  // src/Repository/ProductRepository.php
  namespace App\Repository;

  use App\Entity\Product;
  use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
  use Doctrine\Persistence\ManagerRegistry;

  class ProductRepository extends ServiceEntityRepository
  {
      public function __construct(ManagerRegistry $registry)
      {
          parent::__construct($registry, Product::class);
      }

      public function findExpensiveProducts(float $price)
      {
          return $this->createQueryBuilder('p')
                      ->andWhere('p.price > :price')
                      ->setParameter('price', $price)
                      ->getQuery()
                      ->getResult();
      }
  }
  ```

8. **Theoretical Question:**
- What is the Event Dispatcher in Symfony and what is it used for?
  - The Event Dispatcher is a component in Symfony that allows the dispatching of events and the handling of these events by listeners or subscribers.
  - It is used for implementing the observer pattern, where components can communicate without being tightly coupled.

9. **Code Question:**
- Create a custom validator in Symfony to ensure the email field of a User entity does not belong to a specific domain (e.g., "example.com"). Show how to configure this validator and how it would be used in the User entity.

  ```php
  // src/Validator/Constraints/NoExampleDomain.php
  namespace App\Validator\Constraints;

  use Symfony\Component\Validator\Constraint;

  /**
   * @Annotation
   */
  class NoExampleDomain extends Constraint
  {
      public $message = 'The email "{{ value }}" cannot be from the domain "example.com".';
  }

  // src/Validator/Constraints/NoExampleDomainValidator.php
  namespace App\Validator\Constraints;

  use Symfony\Component\Validator\Constraint;
  use Symfony\Component\Validator\ConstraintValidator;

  class NoExampleDomainValidator extends ConstraintValidator
  {
      public function validate($value, Constraint $constraint)
      {
          /* @var $constraint \App\Validator\Constraints\NoExampleDomain */

          if (null === $value || '' === $value) {
              return;
          }

          if (strpos($value, '@example.com')

 !== false) {
              $this->context->buildViolation($constraint->message)
                  ->setParameter('{{ value }}', $value)
                  ->addViolation();
          }
      }
  }

  // src/Entity/User.php
  namespace App\Entity;

  use Doctrine\ORM\Mapping as ORM;
  use Symfony\Component\Validator\Constraints as Assert;
  use App\Validator\Constraints as AppAssert;

  /**
   * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
   */
  class User
  {
      /**
       * @ORM\Id
       * @ORM\GeneratedValue
       * @ORM\Column(type="integer")
       */
      private $id;

      /**
       * @ORM\Column(type="string", length=100)
       * @Assert\NotBlank
       * @Assert\Email
       * @AppAssert\NoExampleDomain
       */
      private $email;

      // other fields and methods...
  }

  // config/services.yaml
  services:
      App\Validator\Constraints\NoExampleDomainValidator:
          tags: ['validator.constraint_validator']
  ```

10. **Code Question:**
- Implement an Event Subscriber in Symfony that listens to the `kernel.request` event and logs each visit to any page of the application. Ensure the service is configured correctly so that the subscriber is registered with the event.

  ```php
  // src/EventSubscriber/RequestLoggerSubscriber.php
  namespace App\EventSubscriber;

  use Symfony\Component\EventDispatcher\EventSubscriberInterface;
  use Symfony\Component\HttpKernel\Event\RequestEvent;
  use Psr\Log\LoggerInterface;

  class RequestLoggerSubscriber implements EventSubscriberInterface
  {
      private $logger;

      public function __construct(LoggerInterface $logger)
      {
          $this->logger = $logger;
      }

      public function onKernelRequest(RequestEvent $event)
      {
          $request = $event->getRequest();
          $this->logger->info('Page visited: ' . $request->getUri());
      }

      public static function getSubscribedEvents()
      {
          return [
              'kernel.request' => 'onKernelRequest',
          ];
      }
  }

  // config/services.yaml
  services:
      App\EventSubscriber\RequestLoggerSubscriber:
          tags: ['kernel.event_subscriber']
  ```

## Section 2: JavaScript

1. **Theoretical Question:**
- Explain the difference between `var`, `let`, and `const` in JavaScript.
  - `var` is function-scoped and can be re-declared and updated.
  - `let` is block-scoped and can be updated but not re-declared in the same scope.
  - `const` is block-scoped and cannot be updated or re-declared.

2. **Code Question:**
- Write a function in JavaScript that reverses a string.

  ```javascript
  function reverseString(str) {
      return str.split('').reverse().join('');
  }
  ```

3. **Theoretical Question:**
- What is the Event Loop in JavaScript and how does it work?
  - The Event Loop is a mechanism that allows JavaScript to perform non-blocking operations by offloading tasks to the browser's API and processing them later, once the call stack is clear.

4. **Code Question:**
- Write a script in JavaScript that filters even numbers from an array and logs them to the console.

  ```javascript
  const numbers = [1, 2, 3, 4, 5, 6];
  const evenNumbers = numbers.filter(num => num % 2 === 0);
  console.log(evenNumbers);
  ```

5. **Theoretical Question:**
- What is the DOM and how is it manipulated using JavaScript?
  - The DOM (Document Object Model) is a programming interface for HTML documents. It represents the page structure as a tree of nodes. JavaScript can manipulate the DOM by selecting elements and modifying their properties, attributes, and content.

6. **Code Question:**
- Write a JavaScript code that adds an event listener to a button with the id `#myButton` to show an alert with the message "Hello World" when clicked.

  ```javascript
  document.getElementById('myButton').addEventListener('click', function() {
      alert('Hello World');
  });
  ```

7. **Theoretical Question:**
- Explain what a Promise is in JavaScript and provide a basic example.
  - A Promise is an object representing the eventual completion or failure of an asynchronous operation. It allows chaining of operations using `.then()` and handling errors using `.catch()`.

  ```javascript
  let promise = new Promise((resolve, reject) => {
      let success = true;
      if (success) {
          resolve('Operation was successful');
      } else {
          reject('Operation failed');
      }
  });

  promise.then(result => {
      console.log(result);
  }).catch(error => {
      console.log(error);
  });
  ```

8. **Code Question:**
- Write a function in JavaScript that makes an AJAX request (using `fetch`) to get data from an API and displays it in an element with the id `#result`.

  ```javascript
  function fetchData() {
      fetch('https://api.example.com/data')
          .then(response => response.json())
          .then(data => {
              document.getElementById('result').innerText = JSON.stringify(data);
          })
          .catch(error => console.error('Error:', error));
  }
  ```

9. **Theoretical Question:**
- What is the difference between `null`, `undefined`, and `NaN` in JavaScript?
  - `null` is an assignment value representing no value.
  - `undefined` means a variable has been declared but not assigned a value.
  - `NaN` stands for "Not-a-Number" and is a result of invalid or undefined mathematical operations.

10. **Code Question:**
- Implement a function in JavaScript that uses `localStorage` to save a key-value pair and then retrieve it.

  ```javascript
  function saveToLocalStorage(key, value) {
      localStorage.setItem(key, value);
  }

  function getFromLocalStorage(key) {
      return localStorage.getItem(key);
  }

  // Usage
  saveToLocalStorage('name', 'John Doe');
  console.log(getFromLocalStorage('name')); // Outputs: John Doe
  ```

## Section 3: Git

1. **Theoretical Question:**
- What is Git and what is it used for in software development?
  - Git is a version control system used for tracking changes in source code during software development. It allows multiple developers to collaborate on a project, manage versions, and keep track of changes.

2. **Command Question:**
- What is the command to clone a Git repository?

  ```sh
  git clone <repository-url>
  ```

3. **Theoretical Question:**
- Explain what a "branch" is in Git and what it is used for.
  - A branch is a parallel version of a repository. It is used to work on different features or fixes independently without affecting the main codebase.

4. **Command Question:**
- Provide the commands necessary to create a new branch called `feature-xyz`, switch to that branch, and then merge it with the `main` branch.

  ```sh
  git branch feature-xyz
  git checkout feature-xyz
  # Make changes and commit them
  git checkout main
  git merge feature-xyz
  ```

5. **Theoretical Question:**
- What is a "merge conflict" and how is it resolved?
  - A merge conflict occurs when changes in different branches conflict and cannot be automatically merged. It is resolved by manually editing the conflicting files to reconcile the differences and then completing the merge.

6. **Command Question:**
- What is the command to view the current status of the repository in Git?

  ```sh
  git status
  ```

7. **Theoretical Question:**
- Explain the difference between `git pull` and `git fetch`.
  - `git fetch` downloads changes from a remote repository but does not merge them into the local branch. `git pull` fetches changes and immediately merges them into the current branch.

8. **Command Question:**
- What is the command to revert the last commit in Git?

  ```sh
  git revert HEAD
  ```

9. **Theoretical Question:**
- What is a "remote repository" and how is it configured in Git?
  - A remote repository is a version of your project hosted on the internet or another network. It is configured using the `git remote` command.

  ```sh
  git remote add origin <repository-url>
  ```

10. **Command Question:**
- Provide the commands to add all changes to the staging area and then commit them with the message "Initial commit".

  ```sh
  git add .
  git commit -m "Initial commit"
  ```

## Section 4: MySQL

1. **SQL Code Question:**
   Write an SQL query to create a database named company and a table named employees with the following columns: id (INT, auto-increment, primary key), name (VARCHAR(100)), position (VARCHAR(50)), salary (DECIMAL(10, 2)), and hire_date (DATE).

   ```sql
   CREATE DATABASE company;
   USE company;
   CREATE TABLE employees (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100),
       position VARCHAR(50),
       salary DECIMAL(10, 2),
       hire_date DATE
   );
   ```

2. **Theoretical Question:**
   Explain the difference between a primary key (Primary Key) and a foreign key (Foreign Key) in MySQL. When and why are they used?

   - A primary key uniquely identifies each record in a table and ensures data integrity by ensuring that each record has a unique identifier.
   - A foreign key is a field in one table that refers to the primary key in another table, establishing a relationship between the two tables.
   - Primary keys are used to uniquely identify records, while foreign keys are used to establish relationships between tables to enforce referential integrity and maintain data consistency.

3. **SQL Code Question:**
   Write an SQL query to insert three records into the employees table created in question 1.

   ```sql
   INSERT INTO employees (name, position, salary, hire_date)
   VALUES ('John Doe', 'Manager', 55000.00, '2023-01-15'),
          ('Jane Smith', 'Developer', 60000.00, '2022-05-20'),
          ('Mike Johnson', 'Analyst', 65000.00, '2022-09-10');
   ```

4. **SQL Code Question:**
   Show how to update the salary of a specific employee in the employees table. For example, update the salary of the employee with id = 1 to 60000.00.

   ```sql
   UPDATE employees
   SET salary = 60000.00
   WHERE id = 1;
   ```

5. **SQL Code Question:**
   Write an SQL query to select all employees whose salary is greater than 50000.00 and order them by the hire_date field in descending order.

   ```sql
   SELECT *
   FROM employees
   WHERE salary > 50000.00
   ORDER BY hire_date DESC;
   ```

6. **Theoretical Question:**
   What is a transaction in MySQL and how is it used? Provide an example of its usage.

   - A transaction in MySQL is a sequence of SQL statements that are executed as a single unit of work. It ensures that either all the operations within the transaction are completed successfully or none of them are.
   - Transactions are used to maintain data integrity by ensuring that databases remain in a consistent state.
   - Example:
     ```sql
     START TRANSACTION;
     UPDATE account SET balance = balance - 500 WHERE account_id = 123;
     INSERT INTO transaction_log (account_id, amount, transaction_type) VALUES (123, 500, 'withdrawal');
     COMMIT;
     ```

7. **SQL Code Question:**
   Create a view in MySQL named high_earning_employees that selects all columns of employees whose salary is greater than 70000.00.

   ```sql
   CREATE VIEW high_earning_employees AS
   SELECT *
   FROM employees
   WHERE salary > 70000.00;
   ```

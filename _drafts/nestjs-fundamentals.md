# NestJS Fundamentals

NestJS is a Node.js framework aimed at providing a default structure and tools for common applications such as REST APIs, and others. NestJS comes with a CLI tool that helps with a lot of boilerplate activities; to install it:
```shell
npm i -g @nestjs/cli
```

We can use the NestJS CLI to interactively create a new NestJS application:
```shell
nest new
```

this will spawn and interactive application that will ask a bunch of question and then create the new project in a folder with the project name that has been selected. To start the new application, `cd` into the project folder and run:
```shell
npm start
```

This will spawn an HTTP server listening to port 3000: if we navigate to that address, we'll see a simple "Hello, World!" message.

For development purposes, it's usually more useful to run instead:
```shell
npm run start:dev
```

With this command, the server that will be spawned will also provide watch mode, meaning that we won't have to restart the server after every file change.

Every NestJS application is bootstrapped by the `src/main.ts` file, whose purpose is just to create a new NestJS application, and launch it on the specified port. To create an application, a `NestFactory` is used, which is included as part of the NestJS core library. The factory method then needs to take the application module, which is provided by our own application, and defines its features. The root application module is commonly named `AppModule` and defined in the `app.module.ts` file.

Modules are used to bind together related features. Each feature usually needs to react to the user's I/O, perform some logic, and possibly use external resources. In NestJS, all I/O is handled by *controllers*, which are by default REST-based: inputs are then HTTP requests, and outputs are HTTP responses. Application logic and communication with external resources is instead handled by *services*, which are general TypeScript classes that can be injected and used in controllers.

## Controllers

The first step to build a new feature is to generate a controller for it:
```shell
nest generate controller coffees
```

With this command the NestJS CLI automatically adds a new subfolder `src/coffees` containing files for the new controller, and its test: `coffees.controller.ts` and `coffees.controller.spec.ts`. This new controller is also automatically registered with the existing `AppModule`. To create a new controller inside a different folder, we should specify it in the command:
```shell
nest generate controller abc/coffees
```

With the `--dry-run` flag we can see what would be the new files and directory created, without them actually being created.

The new controller has these contents:
```typescript
import { Controller } from '@nestjs/common';

@Controller('coffees')
export class CoffeesController {}
```

The interesting part here is the decorator `@Controller('coffees')`. Here the decorator contains the name of the route that is linked to this controller, in this case `coffees`, meaning that if we navigate to `/coffees`, this is the controller that will handle that request. If we check the predefined `AppController`, in fact, we can see that it uses the empty `@Controller()` decorator, meaning that this decorator responds to requests to `/`.

Having a controller linked to the `/coffees` route is not enough to be able to start responding to requests, though, because we haven't setup any actual action yet. To create an action that responds to a `GET` request, we can add to the controller a method like the following:
```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('coffees')
export class CoffeesController {
  @Get()
  findAll() {
    return 'This action returns all coffees';
  }
}
```

This instructs our controller on what to do when a `GET` request is incoming. The response will by default by `text/html` containing the returned string.

The `@Get()` decorator takes itself a partial route, that will be appended to the one defined by the controller. This means that if we use `@Get('flavors')` instead, this action will be accessed by navigating to `/coffees/flavors` instead.

The `@Get()` decorator can also handle route parameters, like this:
```typescript
import { ..., Param, ... } from `@nestjs/common';
...
@Get(':id')
findOne(@Param('id') id: string) {
  return `This action returns #{id} coffee`;
}
...
```

This action will be triggered by requests to routes like `/coffees/123`, where the number `123` will be assigned to the `id` parameter. We can also get all incoming route parameters by using:
```typescript
...
@Get(':id')
findOne(@Param() params) {
  return `This action returns #${params.id} coffee`;
}
...
```

To handle `POST` requests we need to access also the body of the request this time:
```typescript
import { ..., Post, Body, ... } from `@nestjs/common`;
...
@Post()
create(@Body() body) {
  return body;
}
...
```

Again, we can specify what specific part of the body we're interested in:
```
...
@Post()
create(@Body('name') name: string) {
  return name;
}
...
```

NestJS sends back codes 200 for `GET` and 201 for `POST` by default. We can specify specific status codes for our responses:
```typescript
import { ..., HttpCode, HttpStatus, ... } from '@nestjs/common';
...
@Post()
@HttpCode(HttpStatus.GONE)
create(@Body body) {
  return body;
}
...
```

We can access the actual response object:
```typescript
import { ..., Res, ... } from '@nestjs/common';
...
@Get()
findAll(@Res() response) {
  response.status(200).send('This action returns all coffees');
}
...
```

The actual response object depends on the underlying HTTP library that is being used by NestJS, so we need to consider this when we use that object. By default NestJS is configured to use Express, but this can be replaced by other libraries, like Fastify.

In REST, a `PUT` operation replaces the entire resource, so a complete resource must be present in the body of the request. Instead a `PATCH` operation only replaces a part of the resource:
```typescript
import { ..., Patch, ... } from `@nestjs/common`;
...
@Patch(':id')
update(@Param('id') id: string, @Body() body) {
  return `This action updates #${id} coffee`;
}
...
```

To support the `DELETE` operation:
```typescript
import { ..., Delete, ... } from `@nestjs/common';
...
@Delete(':id')
remove(@Param('id') id: string) {
  return `This action deletes #${id} coffee`;
}
...
```

With the `@Query()` decorator we can access the elements of the request's query string:
```typescript
import { ..., Query, ... } from `@nestjs/common';
...
@Get()
findAll(@Query() paginationQuery) {
  const { limit, offset } = paginationQuery;
  return `This action returns all coffees. Limit: ${limit}, offset: ${offset}`;
}
...
```

So if we send a `GET` request like `/coffees?limit=10&offset=2` our action will be able to extract the `limit` and `offset` values.

## Services

We can scaffold a new service by running
```shell
nest generate service coffees
```

Since a `coffees` folder was already there, NestJS CLI creates the new `CoffeesService` inside that same folder, thus as a sibling of the already existing `CoffeesController`. A test file for the new service will also be created.

The new `CoffeesService` will be registered in `AppModule` as a provider. In NestJS, *providers* are classes that can be injected into other classes, i.e., classes that provide functionality. All providers are thus tagged with the `@Injectable()` decorator:
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class CoffeesService {
  private coffees = [];
}
```

We can inject our new `CoffeesService` into our `CoffeesController`:
```typescript
...
import { CoffeesService } from './coffees.service';
...
constructor(private readonly coffeesService: CoffeesService) {}
```

We can use `HttpExceptions` to cause an HTTP error to be returned to the user, for example:
```typescript
import { ..., HttpException, HttpStatus, ... } from '@nestjs/common';
...
const coffee =  this.coffees.find(item => item.id === +id);
if (!coffee) {
  throw new HttpException(`Coffee #${id} not found`, HttpStatus.NOT_FOUND);
}
return coffee;
...
```

Most common error situations have their own dedicate exceptions. For example, `HttpException` with `HttpStatus.NOT_FOUND` can be replaced with `NotFoundException`:
```typescript
import { ..., NotFoundException, ... } from '@nestjs/common';
...
throw new NotFoundException(`Coffee #${id} not found`);
...
```

NestJS has a built-in exception handling layer, so if we don't manually catch some exception, they'll still be caught and transformed into nicer HTTP exceptions:
```typescript
...
throw 'A random error';

```

This will be tranformed by NestJS into a `500` HTTP error with a generic message, to avoid discosing any implementation information, and the server log (that we can check in the terminal where we're running the application) will contain the actual exception message.

## Domain

Related features should be kept together into specific modules:
```shell
nest g module coffees
```

This will create a new `coffees.module.ts` inside the already existing `coffees` folder. The new module is bootstrapped as:
```typescript
import { Module } from '@nestjs/common';

@Module({})
export class CoffeesModule {}
```

Inside the `@Module()` decorator we can set these configurations:
- `controllers` the controllers that are defined in this module, and that should be instantiated, in order for them to be able to handle user input
- `exports` the providers defined in this module that should be available anywhere this module is imported
- `imports` the external modules that should be visible from this module, in order for their providers to be able to be used from within this module
- `providers` the classes defined in this module that should be configured with the dependency injection container to be able to be injected

The `controllers` and `providers` settings make *definitions*, meaning that they cause classes to be instantiated. When it comes to services and controllers, it's important that they're never instantiated twice, so we must be careful to never add the definitions of the same controllers or services to more than one module. For example, if we want `CoffeesController` and `CoffeesService` to belong to `CoffeesModule` instead of `AppModule` we need not only to add them to `controllers` and `providers` of `CoffeesModule`:
```typescript
import { Module } from '@nestjs/common';
import { CoffeesController } from './coffees.controller';
import { CoffeesService } from './coffees.service';

@Module({
  controllers: [ CoffeesController ],
  providers: [ CoffeesService ],
})
export class CoffeesModule {}
```

but also to remove those definitions from `AppModule`.

Data that is moved around from user to controllers, or from controllers to services and vice-versa, is usually wrapped in DTOs, Domain Transfer Objects, in order to assign a type to relevant groups of data:
```shell
nest g class coffees/dto/create-coffee.dto --no-spec
```

DTOs are conventionally stored inside a `dto` folder in the module, with files having a `.dto` suffix. Also, since DTOs are nothing more than bare container classes, there's no need to write tests for them. This command will create an empty class named `CreateCoffeeDto`.

Additionally, DTOs are conventionally named after their expected usage. For example, `CreateCoffeeDto` is meant to be used to contain the data needed to create a new `Coffee`:
```typescript
export class CreateCoffeeDto {
  readonly name: string;
  readonly brand: string;
  readonly flavors: string[];
}
```

We can thus use this brand new DTO in the `create()` mtehod of the `CoffeeController`:
```typescript
...
import { CreateCoffeeDto } from './dto/create-coffee.dto';
...
@Post()
create(@Body() createCoffeeDto: CreateCoffeeDto) {
  return this.coffeesService.create(createCoffeeDto);
}
```

For `PATH` operations we should make all DTO properties optional, because we might want to change only some values of the entity:
```typescript
export class UpdateCoffeeDto {
  readonly name?: string;
  readonly brand?: string;
  readonly flavors?: string[];
}
```

NestJS provides a way to automatically validate DTOs that are coming from the user. In order to use this, we need to make this change to `main.ts`:
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

Here we added the line `app.useGlobalPipes(new ValidationPipe());`, and the required import. In order for this code to work, we also need to add the following packages:
```shell
npm i class-validator class-transformer
```

From now on, we can add validators to our DTOs. For example:
```typescript
import { IsString } from 'class-validator';

export class CreateCoffeeDto {
  @IsString()
  readonly name: string;

  @IsString()
  readonly brand: string;

  @IsString({ each: true })
  readonly flavors: string[];
}
```

The first consequence of this is that now each of these properties is required. Before adding the validators, we could send a `POST` request to create a new `Coffee`, with a payload that didn't match exactly our DTO, and the application wouldn't complain. Now, if we try to send an incomplete payload, we receive a `400` error, containing a detailed description of all the fields that are invalid or missing.

NestJS provides a tool to help avoid redundancy in create DTOs and update DTOs. This tool is available from:
```shell
npm i @nestjs/mapped-types
```

With this package installed, instead of re-defining the `UpdateCoffeeDto`, we can do the following:
```typescript
import { PartialType } from '@nestjs/mapped-types';
import { CreateCoffeeDto } from './create-coffee.dto';

export class UpdateCoffeeDto extends PartialType(CreateCoffeeDto) {}
```

The `PartialType` class just returns the class that has been passed as argument, in this case `CreateCoffeeDto`, with all properties set to optional. This way, we can now update `CreateCoffeeDto` with new properties, avoiding the risk of forgetting to subsequentially update `UpdateCoffeeDto` each time. Additionally, `PartialType` inherits all the validation rules set to `CreateCoffeeDto`, which are thus now applied to `UpdateCoffeeDto`, and also add the `IsOptional()` rule to each field, to allow them to be optional.

With the current setup, however, we're still able to pass additional properties to our methods, that are not part of our expected DTOs. We can configure the `ValidationPipe` so that any payload property that is not recognized is stripped away. Head over to `main.ts` and apply the following change:
```typescript
...
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
}));
...
```

We can also return an error if any invalid property is passed to the payload:
```typescript
...
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
}));
...
```

Payload data that is passed over the network are plain JavaScript objects for performance reasons. If they have the same structure of our expected DTOs, they can be used as such. However, we can make it so these objects are actually transformed in the exact DTOs we have defined:
```typescript
...
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
}));
...
```

This way, from inside the `create()` method of our controller, the `createCoffeeDto` object will exactly be an instance of `CreateCoffeeDto`, instead of a plain class that happens to have the same properties. The `transform` feature also converts all primitive types of the request to the expected ones. For example, if we apply the type `number` to the `id` parameter of the `findOne()` method, instead of `string`, NestJS will try to convert the route parameter that we used to a number.

## Database integration

In the TypeScript world we can use the TypeORM ORM to work with databases. To use TypeORM with a PostgreSQL:
```shell
npm i @nestjs/typeorm typeorm pg
```

Next, edit `AppModule` to import the external TypeORM module:
```typescript
...
import { TypeOrmModule } from '@nestjs/typeorm';
...

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'pass123',
      database: 'postgres',
      autoLoadEntities: true,
      synchronize: true,
    }),
    ...
    }
  ],
  ...
})
...
```

The `autoLoadEntities` option allows to load entities automatically instead of having to manually specify them. The `synchronize` option makes sure that the ORM is synchronizes with the database every time we run our application, but should not be used in production.

An *entity* represents a relation between a TypeScript class and a database table. The previous `synchronize` option of the `TypeOrmModule` setup has the effect that every time the application is run the ORM checks if all entities defined in the application with the `@Entity()` decorator have a matching table in the database, and if not, it creates all the missing ones:
```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class Coffee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  brand: string;

  @Column('json', { nullable: true })
  flavors: string[];
}
```

By default TypeORM will generate SQL tables having the same name of the related class, in lowercase, so in this case the `coffee` table will be generated. We can select a different name by passing it to the decorator: `@Entity('coffees')`. The `Column()` decorator is used to select a property to be used as a table column in the database. The `PrimaryGeneratedColumn` decorator is used to select a property as the primary key of the table, and also to let its values be automatically generated by the database. Since the last property, `flavors` is an array, we need to tell TypeORM how to store this complex data: in this case, we specified that the data should be stored as a JSON string, and that the column should be nullable. This also means that all other columns, not having the `nullable: true` configuration, are instead required.

Once the entity has been defined, we need to register it to TypeORM. The first thing to do is to import the module inside the `CoffeeModule` as well:
```typescript
...
import { TypeOrmModule } from '@nestjs/typeorm';
import { Coffee } from './entities/coffee.entity';
...
@Module({
  imports: [TypeOrmModule.forFeature([Coffee])],
})
...
```

The TypeORM module should be imported only once `forRoot()` in the root module (the `AppModule`) in order to set up its global configuration, and once for each module that needs to use database features, but this time `forFeature()`. In this occasion, we register all the entities that should be handled by TypeORM in that module.

In NestJS each entity that is defined has its own repository, which abstracts the access to the data source. For example, we can inject this repository (that is automatically generated once the entity is defined) into our `CoffeeService`:
```typescript
...
import { InjectRepository } form '@nestjs/typeorm';
import { Repository } from 'typeorm';
...

constructor(@InjectRepository(Coffee) private readonly coffeeRepository: Repository<Coffee>) {}
...
```

We can also remove the property `coffees` that was containing mock data. Since now we're going to read from an actual external service, some communications need to become asynchronous:
```typescript
...
import { CreateCoffeeDto } from './dto/create-coffee.dto';
import { UpdateCoffeeDto } from './dto/update-coffee.dto';
...
findAll() {
  return this.coffeeRepository.find();
}

async findOne(id: string) {
  const coffee = await this.coffeeRepository.findOne(id);
  if (!coffee) {
    throw new NotFoundException(`Coffee #${id} not found`);
  }
  return coffee;
}

create(createCoffeeDto: CreateCoffeeDto) {
  const coffee = this.coffeeRepository.create(createCoffeeDto);
  return this.coffeeRepository.save(coffee);
}

async update(id: string, updateCoffeeDto: UpdateCoffeeDto) {
  const coffee = await this.coffeeRepository.preload({
    id: +id,
    ...updateCoffeeDto,
  });
  if (!coffee) {
    throw new NotFoundException(`Coffee #${id} not found`);
  }
  return this.coffeeRepository.save(coffee);
}

async remove(id: string) {
  const coffee = await this.findOne(id);
  return this.coffeeRepository.remove(coffee);
}
...
```

Every method that needs to retrieve data from the database needs to deal with asynchronous communication: `findAll` will return a promise, `findOne()` needs to wait for the result before checking if it's valid, `update` needs again to wait for the repository to return an entity before checking it, and `remove` needs to wait to see if the requested entity actually exists, before attempting at removing it (additionally, the return value of `coffeeRepository.remove()` is again a promise).

The `this.coffeeRepository.preload()` call creates a new entity based on the object passed into it: if a matching entity already exists in the database, it replaces everything with the data passed as argument, and then return the result; if the `id` of the entity passed into the method does not match any entity saved in the database, the method will return `undefined`.

Entities of different kinds ar every often linked to each other by *relations*, which are associations between common fields of the entities. One-to-one relations are such that every row in the primary table has one, and only one, associated row in the foreign table. One-to-many or many-to-one relations are such that every row in the primary table has one or more associated rows in the foreign table. Many-to-many relations are such that every row in the primary table has many related rows in the foreign table, and every row in the foreign table has many related rows in the primary table.

To define a relation in one of our entity:
```typescript
import { ..., JoinTable, ManyToMany, ... } from 'typeorm';
import { Flavor } from './flavor.entity';
...
@JoinTable()
@ManyToMany(type => Flavor, flavor => flavor.coffees)
flavors: string[]
```

The `@JoinTable()` decorator specifies that the decorated field has to be populated with data taken from another table that should be joint to this one. This decorator needs the presence of another decorator to further specify the nature of the relation between the two tables. In this case we added the `@ManyToMany()` decorator, to define a many-to-many relation. The first argument is a callback that returns a reference to the related entity, in this case the entity `Flavor`; the second argument is another callback that receives a row of the related entity, from which we need to select which of its fields is the reverse side of the relation, pointing back to the current entity. The `@JoinTable()` decorator has to be added only to one side, the owning side, of the relation, meaning that in our model `Coffee`'s have `Flavor`'s, and not vice-versa.

We need then a companion `Flavor` entity defined in such a way that the relation with `Coffee` holds:
```typescript
import { Column, Entity, ManyToMany, PrimaryGeneratedColumn } from 'typeorm';
import { Coffee } form './coffee.entity';

@Entity()
export class Flavor {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(type => Coffee, coffee => coffee.flavors)
  coffees: Coffee[];
}
```

Here we see that, in addition to define standard fields for the `id` and `name` of the flavors, we also added a field that points back to the `coffees` that have the current flavor. Since this is again a many-to-many relation, we need to add a `ManyToMany()` decorator to the field, specifying again the entity that we are pointing to, `Coffee`, and the field in that entity that points back to us, `coffee.flavors`. Additionally, since this is not the owning side of the relation, we should not add the `JoinTable()` decorator.

Once we add the new entity `Flavor` to the `CoffeesModule`:
```typescript
...
import { Flavor } form './entities/flavor.entity';
...
@Module({
  imports: [ TypeOrmModule.forFeature([Coffee, Flavor])],
  ...
})
...
```

the PostgreSQL database will be synchronized with two new tables: `flavor` to contain the actual data of the `Flavor` entities, and `coffee_flavors_flavor` to store the many-to-many relations between `Coffee` and `Flavor`.

When we retrieve entities from a repository, relations are not eagerly loaded, for performance reasons. This means that if we want to show the flavors of each coffee, when we retrieve a coffee for the API, we need to explicitly load them:
```typescript
...
findAll() {
  return this.coffeeRepository.find({
    relations: ['flavors'],
  });
}

async findOne(id: string) {
  const coffee = await this.coffeeRepository.findOne(id, {
    relations: ['flavors'],
  });
  ...
}
...
```

Here we just configured the repository to add the specified relations to the output. The relations are the names of the fields of the `Coffee` entity that are defined as relations with other tables.

At this point we need to figure out how to insert new flavors in their own table, as well as their relations with the containing coffees. When a user adds a new coffee, they'll provide flavors together with the other coffee data, and not as separate payloads. We could then think of separating the flavors from the coffee data, and make an additional insert operation, to add the new flavors to the repository, before creating the relations. However, TypeORM provides a better solution for this, which is *cascading inserts*:
```typescript
...
@JoinTable()
@ManyToMany(
  type => Flavor,
  flavor => flavor.coffees,
  {
    cascade: true,
  }
)
...
```

Here we're specifying that every time a record is created or updated, the corresponding record in the related table must be created or updated too. We could also have specified the cascading behavior only for insert or update:
```typescript
...
{
  cascade: ['insert'],
}
...
```

Next, we need to handle `Flavor`'s in the `CoffeeService`:
```typescript
...
import { Flavor } from './entities/flavor.entity';
...
constructor(
  ...
  @InjectRepository(Flavor)
  private readonly flavorRepository: Repository<Flavor>,
) {}
...
async create(createCoffeeDto: CreateCoffeeDto) {
  const flavors = await Promise.all(
    createCoffeeDto.flavors.map(name => this.preloadFlavorByName(name)),
  );

  const coffee = this.coffeeRepository.create({
    ...createCoffeeDto,
    flavors,
  });
  return this.coffeeRepository.save(coffee);
}
async update(id: string, updateCoffeeDto: UpdateCoffeeDto) {
  const flavors = updateCoffeeDto.flavors && (await Promise.all(
      updateCoffeeDto.flavors.map(name => this.preloadFlavorByName(name)),
  ));

  const coffee = await this.coffeeRepository.preload({
    id: +id,
    ...updateCoffeeDto,
    flavors,
  });
  if (!coffee) {
    throw new NotFoundException(`Coffee #${id} not found`);
  }
  return this.coffeeRepository.save(coffee);
}
...
private async preloadFlavorByName(name: string): Promise<Flavor> {
  const existingFlavor = await this.flavorRepository.findOne({ name });
  if (existingFlavor) {
    return existingFlavor;
  }
  return this.flavorRepository.create({ name });
}
```

When we create a new `Coffee`, we need to make sure that the `flavors` of the DTO are populated as well: to do this, we first get from the `flavorRepository` the `Flavor`'s matchine the ones passed in as strings, creating them anew if necessary. Then we just merge these flavors with the object used to create a new `Coffee`. In order for this to work, we need to update `Coffee` so that `flavors` is actually `Flavor[]` instead of `string[]`:
```typescript
...
@JoinTable()
@ManyToMany(
  type => Flavor,
  flavor => flavor.coffees,
  {
    cascade: true,
  },
)
flavors: Flavor[];
```

A similar thing is done for the `update` method of the service. The only difference is that in this case `updateCoffeeDto.flavors` might not be present, because in the update operations the fields are optional, so we need to first check if the user is trying to update the flavors as well, before we head over to the flavors repository to get actual `Flavor` entities.

We can use our TypeORM also to handle pagination, meaning to retrieve only the portion of data that the user requested. First, we need to create a DTO to hold pagination options:
```shell
nest g class common/dto/pagination-query.dto --no-spec
```

We can then add the paginagion properties to this class:
```typescript
import { Type } from 'class-transformer';
import { IsOptional, IsPositive } from 'class-validator';

export class PaginationQueryDto {
  @IsOptional()
  @IsPositive()
  @Type(() => Number)
  limit: number;

  @IsOptional()
  @IsPositive()
  @Type(() => Number)
  offset: number;
}
```

Query parameters are sent to the network as strings, so we need to transform them to numbers if we want to treat them as such. We can do this manually by using the `@Type()` decorator applied to the DTO fields representing query parameters. Alternatively, we can enable query parameters to be always transformed globally:
```typescript
...
async function bootstrap() {
  ...
  app.useGlobalPipe(
    new ValidationPipe({
      ...
      transformOptions: {
        enableImplicitConversion: true,
      },
    }),
  );
}
...
```

To use our new `PaginationQueryDto`, we just need to assign it as the type of the query argument in our controller:
```typescript
...
import { PaginationQueryDto } from '../common/dto/pagination-query.dto';
...
@Get()
findAll(@Query() paginationQuery: PaginationQueryDto) {
  return this.coffeesService.findAll(paginationQuery);
}
```

And in our service:
```typescript
...
import { PaginationQueryDto } from '../common/dto/pagination-query.dto';
...
findAll(paginationQuery: PaginationQueryDto) {
  const { limit, offset } = paginationQuery;
  return this.coffeeRepository.find({
    relations: ['flavors'],
    skip: offset,
    take: limit,
  });
}
```

Here, in addition to updating the argument type, we're also applying our pagination data to our repository query. In particular, the `skip` option of the `find()` method contains the number of entities to skip before starting to take those to return, or in other words, the results offset; the `take` option contains the number of entities to take to build the results collection.

It may happen that a single user interaction with the application needs to trigger two or more changes to the database, which need to be consistent, meaning that if even one of them fails, all of them need to be reverted. In these cases we need to reason in terms of transactions: a *transaction* is a unit of work, which consists of different basic database operations that either are all correctly performed, or none of them is.

To create transactions, we first need to inject the `Connection` service in our service:
```typescript
...
import { ..., Connection } from 'typeorm';
import { Event } from '../events/entities/event.entity';
...
constructor(
  ...,
  private readonly connection: Connection,
) {}
...
async recommendCoffee(coffee: Coffee) {
  const queryRunner = this.connection.createQueryRunner();

  await queryRunner.connect();
  await queryRunner.startTransaction();
  try {
    coffee.recommendations++;

    const recommendEvent = new Event();
    recommendEvent.name = 'recommend_coffee';
    recommendEvent.type = 'coffee';
    recommendEvent.payload = { coffeeId: coffee.id };

    await queryRunner.manager.save(coffee);
    await queryRunner.manager.save(recommendEvent);

    await queryRunner.commitTransaction();
  } catch (err) {
    await queryRunner.rollbackTransaction();
  } finally {
    await queryRunner.release();
  }
}
```

The core of transactions handling is the usage of query runners. The `Connection` service represents the current connection of our application to the database. We then create a new query runner for this connection with `this.connection.createQueryRunner()`: this is a service that helps us running generic queries on the database. A query can be in the form of a transaction, for which we need to prepare our query runner with `await queryRunner.connect()` and `await queryRunner.startTransaction()`. At this point we can prepare the entities we want to add or update in the database, and call `await queryRunner.manager.save()` to save them first, and finally call `await queryRunner.commitTransaction()` to actually apply the changes to the real database. When we have started a transaction, all database operations are not actually performed the moment they're called, but rather they're saved in the transaction, and really applied only when the transaction is committed. We're wrapping all this code inside a `try/catch` block so that, if an error occurs, we can discard the changes that we were planning to do with `await queryRunner.rollbackTransaction()`. Either case, we need to release the query runner at the end with `await queryRunner.release()`.

Indexes are special lookup tables that the database can use to speed searches. Let's pretend a very common search in our application is looking for an event based on its name. To speed up this kind of search, we can create an index on the `name` column of the `Event` entity:
```typescript
...
import { ..., Index } from 'typeorm';
...
@Index()
@Column()
name: string;
...
```

We can define a composite index by applying the `@Index()` decorator to the entire entity class instead of the single property, and passing an array of field names that make up the composite index:
```typescript
...
@Index(['name', 'type'])
export class Event {
  ...
}
```

TypeORM can be configured in a more solid way by using a `ormconfig.js` file, located at the project root:
```javascript
module.exports = {
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'pass123',
  database: 'postgres',
  entities: ['dist/**/*.entity.js'],
  migrations: ['dist/migrations/*.js'],
  cli: {
    migrationsDir: 'src/migrations',
  },
};
```

In addition to the connection parameters we previosly added to the `AppModule`, we are also adding here the locations where the entities and migrations files are located. The TypeORM configuration is totally external to NestJS, and also to TypeScript, which is why we need to refer to the build directory `dist` instead of the `src` directory.

With this configuration in place, we can create our first migration using TypeORM's CLI tool:
```shell
npx typeorm migration:create -n CoffeeRefactor
```

This creates a new migration file inside `src/migrations`:
```typescript
import {MigrationInterface, QueryRunner} from "typeorm";

export class CoffeeRefactor1610445346676 implements MigrationInterface {

    public async up(queryRunner: QueryRunner): Promise<void> {
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
    }
}
```


Now, let's pretend we need to update the field `name` of `Coffee` to `title`:
```typescript
...
@Column()
title: string;
```

Since we are working with `synchronize: true`, our local development database will be automatically updated as soon as we save our file and the server has restarted. But in order to be able to also update the production database, we need to update the database schema. Changing a column name means removing a column and creating another one, which in turn means that all the data originally located in the column will be lost. What we can do, then, is to instruct our migration to store the original column data, remove the column, create the new one, and restore the saved data:
```typescript
...
public async up(queryRunner: QueryRunner): Promise<any> {
  await queryRunner.query(
    `ALTER TABLE "coffee" RENAME COLUMN "name" TO "title"`,
  );
}

public async down(queryRunner: QueryRunner): Promise<any> {
  await queryRunner.query(
    `ALTER TABLE "coffee" RENAME COLUMN "title" TO "name"`,
  );
}
```

In order for the TypeORM CLI to be able to find the files it needs, we need to make sure that the `dist` folder is up to date:
```shell
npm run build
```

Now we're set to run the actual migration:
```shell
npx typeorm migration:run
```

To revert a migration:
```shell
npx typeorm migration:revert
```

The revert operation cancels the last migration that has been performed, thus calling its `down()` method. If we call `revert` multiple times, we'll be reverting migrations backwards in the order they've been created.

The most common scenario with migrations, though, is to add some definitions to entities, and then create a migration based on the differences between the definitions and the current schema. For this to work, however, we first need to put `synchronize: false` in `AppModule`, otherwise the database will be immediately updated after our changes, and the migration tool won't see any difference. After we've disabled synchronization, let's add a new column:
```typescript
...
@Column({ nullable: true })
description: string;
```

Then we compile the code with `npm run build`, and we generate a new migration with:
```shell
npx typeorm migration:generate -n SchemaSync
```

With `migration:create` we just create an empty migration, that we must then fill in by hand. With `migration:generate`, instead, we let the migration tool check the differences between our entities definition and the current database schema, and generate a migration for us:
```typescript
import {MigrationInterface, QueryRunner} from "typeorm";

export class SchemaSync1610454724344 implements MigrationInterface {
    name = 'SchemaSync1610454724344'

    public async up(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.query(`ALTER TABLE "coffee" ADD "description" character varying`);
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.query(`ALTER TABLE "coffee" DROP COLUMN "description"`);
    }
}
```

At this point, we can apply this migration to get our database up to date:
```shell
npx typeorm migration:run
```

## Dependency Injection

In NestJS, the NestJS runtime environment works also as a dependency injection container. When the container instantiates the `CoffeesController`, it checks if there's any dependencies needed. In our case there's one dependency: `coffeesService` of type `CoffeesService`. Given the type of the dependency, the container will check if an instance of `CoffeesService` is cached, and use that one, or it creates one and caches it. This is because by default the container is configured to use singletons. When creating a new instance of `CoffeesService`, again the container first checks if that class has dependencies, and it resolves each of them beforehand. All this analysis and instantiation is done during the bootstrap of the application.

When we register a provider with a module, what we're doing is this:
```typescript
...
@Module({
  ...
  providers: [
    {
      provide: CoffeesService,
      useClass: CoffeesService,
    }
  ]
})
...
```

Here the `provide` option is the name we're giving to this provider to identify it, and `useClass` is the actual class that needs to be instantiated. In the most common scenario, we just say `providers: [CoffeesService]`, thus using the class name as provider identifier as well.

Let's create a new module:
```shell
nest g mo coffee-rating
```

and service within that module:
```shell
nest g s coffee-rating
```

Our new module depends on the existing `CoffeesService`, so we need to import `CoffeesModule` into `CoffeeRatingModule`:
```typescript
...
import { CoffeesModule } from '../coffees/coffees.module';
...
@Module({
  imports: [ CoffeesModule ],
  ...
})
...
```

And the `CoffeeRatingService` needs the `CoffeesService`:
```typescript
...
import { CoffeesService } from '../coffees/coffees.service';
...
constructor( private readonly coffeesService: CoffeesService) {}
```

After doing this, we'll see an error in the server's log:
```
Error: Nest can't resolve dependencies of the CoffeeRatingService (?). Please make sure that the argument CoffeesService at index [0] is available in the CoffeeRatingModule context.
```

What's happening is that we're trying to use the `CoffeesService` defined in `CoffeesModule` from `CoffeeRatingModule`, but `CoffeesModule` is not exporting `CoffeesService`, meaning that it's not making it visible from the outside. To fix this:
```typescript
...
@Module({
  ...
  exports: [ CoffeesService ]
})
...
```

It often happens that we need to use some more complex dependency injection logic, for example if we define our dependencies as interfaces, and then we need to tell the container which concrete instances to inject in each case. To do this we need to use *custom providers*.

The easiest form of custom provider is the one using the `useValue` syntax:
```typescript
...
providers: [
  {
    provide: CoffeesService,
    useValue: new MockCoffeesService()
  }
]
...
```

With this configuration, we're providing the container with the exact object to be always injected everywhere the `CoffeesService` dependency is required.

We can use `useValue` providers also with strings, symbols, etc.:
```typescript
...
providers: [
  {
    provide: 'COFFEE_BRANDS',
    useValue: ['buddy brew', 'nescafe'],
  }
]
...
```

To access this new provider:
```typescript
...
import { ..., Inject } from '@nestjs/common';
...
constructor(
  @Inject('COFFEE_BRANDS') coffeeBrands: string[],
) { }
...
```

The `useClass` syntax allows us to tell the container the exact class to instantiate when a dependency with a certain name is requested:
```typescript
...
providers: [
  {
    provide: ConfigService,
    useClass: process.env.NODE_ENV === 'development' ? DevelopmentConfigService : ProductionConfigService,
  }
]
...
```

Another custom provider type is the `useFactory` provider, to create providers dynamically:
```typescript
...
providers: [
  {
    provide: COFFEE_BRANDS,
    useFactory: () => [ 'buddy brew', 'nescafe' ],
  }
]
...
```

With providers factories we can also inject other providers inside the factory:
```typescript
...
@Injectable()
export class CoffeeBrandsFactory {
  create() {
    return ['buddy brew', 'nescafe'];
  }
}
...
providers: [
  {
    provide: COFFEE_BRANDS,
    useFactory: (brandsFactory: CoffeeBrandsFactory) => brandsFactory.create(),
    inject: [CoffeeBrandsFactory],
  }
]
...
```

We can delay the injection after a certain process has completed, by using asynchronous providers:
```typescript
...
providers: [
  {
    provide: COFFEE_BRANDS,
    useFactory: async (connection: Connection): Promise<string[]> => {
      const coffeeBrands = await Promise.resolve(['buddy brew', 'nescafe']);
      return coffeeBrands;
    },
    inject: [Connection],
  }
]
...
```

Let's suppose we have a generic module that needs to behave differently in different circumstances, and in particular it needs to retrieve some configuration before it can be consumed:
```shell
nest g mo database
```
```typescript
import { Module } from '@nestjs/common';
import { createConnection } from 'typeorm';

@Module({
  providers: [
    {
      provide: 'CONNECTION',
      useValue: createConnection({
        type: 'postgres',
        host: 'localhost',
        port: 5432,
      }),
    },
  ],
})
export class DatabaseModule {}
```

Now, what if we want to reuse this module from a different application that needs to use it with a different configuration? To achieve this we need to create a *dynamic module* instead:
```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { createConnection, ConnectionOptions } from 'typeorm';

@Module({})
export class DatabaseModule {
  static register(options: ConnectionOptions): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [
        {
          provide: 'CONNECTION',
          useValue: createConnection(options),
        }
      ]
    }
  }
}
```

Now we can import this module from another module like this:
```typescript
...
imports: [
  DatabaseModule.register({
    type: 'postgres',
    host: 'localhost',
    password: 'password',
    port: 5432,
  }),
  ...
]
...
```

In NestJS almost everything is shared across incoming requests. In NodeJS every request is processed by a separate thread: for this reason we can reuse the same (singleton) instance of services for each request. However, in certain cases we need to spawn a new instance of a provider for every new request incoming. In NestJS the default container scope is one where providers are instantiated as singletons. We could specify this explicitly:
```typescript
...
import { ..., Scope } from '@nestjs/common';
...
@Injectable({ scope: Scope.DEFAULT })
...
```

Another scope for the container is "transient": in this case each consumer that requires a provider will receive a new dedicated instance of that provider. The instantiation of services is still performed once during the application bootstrap, but this time if a provider is requested by more than one consumer, then it's instantiated once per each consumer. This behavior is activated with `@Injectable({ scope: Scope.TRANSIENT })`.

With the "request" scope, a new instance of the provider is created for each new incoming request, but no instance is created at bootstrap. Additionally, all services that use this provider must also be created with the request scope: if they were created at bootstrap, they wouldn't have the dependency they need injected, since it will only be instantiated at the time of the first request. The new instance is also automatically garbage collected as soon as the request has completed processing. To activate this behavior use `@Injectable({ scope: Scope.REQUEST })`. We can also access the original request that triggered the instantiation of this provider by injecting it with `@Inject(REQUEST) private readonly request: Request`, where `REQUEST` comes from `@nestjs/core` and `Request` comes from `express`.

## Configuration

We usually need to configure our application slightly differently according to the actual environment we're running it in. NestJS comes with a configuration package providing a lot of useful tools for handling configuration:
```shell
npm i @nestjs/config
```
```typescript
...
import { ConfigModule } from '@nestjs/config';
...
imports: [
  ...
  ConfigModule.forRoot(),
  ...
]
...
```

Here, `ConfigModule.forRoot()` is the usual method that should be called to import a module in the root module `AppModule`, and loads the configuration from the default location, which is the `.env` file in the project's root directory.
```shell
DATABASE_USER=postgres
DATABASE_PASSWORD=pass123
DATABASE_NAME=postgres
DATABASE_PORT=5432
DATABASE_HOST=localhost
```

NestJS configuration will automatically read the current `.env` file, and load its key-value pairs into the current process' environment. Thus we can retrieve those configurations from the `process.env` object:
```typescript
...
TypeOrmModule.forRoot({
  type: 'postgres',
  host: process.env.DATABASE_HOST,
  port: +process.env.DATABASE_PORT,
  username: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
  ...
})
...
```

We can get the configuration file from another location:
```typescript
...
ConfigModule.forRoot({ envFilePath: ['.environment', '.other']}),
...
```

If the same variable is defined in multiple configuration files, the first instance takes precedence over the rest. It'd be useful if we could validate the schema of the configuration variables we need in our application, in order to throw an error if some needed configurations are not present. To create validation schemas for objects we can use the Joi package:
```shell
npm i @hapi/joi
npm i --save-dev @types/hapi__joi
```

Then we can define a validation schema for the configuration:
```typescript
...
import * as Joi from '@hapi/joi';
...
ConfigModule.forRoot({
  validationSchema: Joi.object({
    DATABASE_HOST: Joi.required(),
    DATABASE_PORT: Joi.number().default(5432),
  }),
}),
...
```

To read configuration variables throughout the application, we can make use of the `ConfigService`, provided we imported the `ConfigModule` in the module where we want to use it:
```typescript
...
import { ConfigModule } from '@nestjs/config';
...
imports: [ ..., ConfigModule ],
...
```
```typescript
...
import { ConfigService } from '@nestjs/config';
...
constructor(
  ...
  private readonly configService: ConfigService,
) {
  const databaseHost = this.configService.get<string>('DATABASE_HOST', 'localhost');
}
...
```

The second argument to `get()` is the default value to use if we don't set the requested configuration.

We can compose our configuration from different configuration files. For example, we can have a `config/app.config.ts` with generic application configuration:
```typescript
export default () => ({
  environment: process.env.NODE_ENV || 'development',
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432
  }
});
```

This configuration file can return any JavaScript object, and as such we can add any logic we need to produce the desired values. Then we can load this file from the config module setup:
```typescript
...
import appConfig from './config/app.config';
...
ConfigModule.forRoot({
  load: [appConfig]
}),
...
```

The new values we provided from the additional configuration are now available from the configuration service:
```typescript
...
constructor(
  ...
  private readonly configService: ConfigService,
) {
  const databaseHost = this.configService.get('database.host', 'localhost');
}
...
```

The dot-notation in `database.host` is a path that instructs the configuration service to navigate through the configuration values, in this case telling to first get the `database` key from the configuration, and then the `host` key from the resulting object.

Let's say we have a configuration specific to a module, `coffees/config/coffees.config.ts`:
```typescript
import { registerAs } from '@nestjs/config';

export default registerAs('coffees', () => ({
  foo: 'bar',
}));
```

Here `registerAs()` allows to register a configuration namespace under the provided key, `coffees` in this case. We can then register the configuration file with `CoffeesModule`:
```typescript
...
import coffeesConfig from './config/coffees.config';
...
imports: [
  ...
  ConfigModule.forFeature(coffeesConfig),
],
...
```

This is known as *partial registration*, since we're registering portions of configuration that are only module-specific, instead of putting everything in the global configuration. Of course these can still be retrieved from the configuration service:
```typescript
...
constructor(
  ...
  private readonly configService: ConfigService,
) {
  const coffeesConfig = this.configService.get('coffees');
  const value = this.configService.get('coffees.foo');
}
...
```

So here we can simply pass the namespace we want to retrieve configurations from, or using the dot-notation to retrieve specific values.

Instead of retrieving the namespace from the config service, we can directly inject the namespace itself:
```typescript
...
import { ..., Inject } from '@nestjs/common';
import { ..., ConfigType } from '@nestjs/config';
import coffeesConfig from './config/coffees.config';
...
constructor(
  ...
  @Inject(coffeesConfig.KEY)
  private readonly coffeesConfiguration: ConfigType<typeof coffeesConfig>,
) {
  const value = coffeesConfiguration.foo;
}
...
```

This way we additionally have the benefit of type checks for our configuration, instead of having to rely on the dot-notation.

Loading configuration from files if of course an I/O operation, that is best handled asynchronoulsy:
```typescript
...
imports: [
  TypeOrmModule.forRootAsync({
    useFactory: () => ({
      type: 'postgres',
      ...
    }),
  }),
],
...
```

Modules setup that are marked as asynchronous will be performed after every other non-asynchronous setup is completed, meaning also after the configuration files have been read.

## Exceptions filters

NestJS has a global exceptions layer that catches any exception that has not been caught by any other part of the application. In particular, this action is performed by the built-in global exceptions filter. However, we can customize the way these exceptions are handled. For example, let's say we want to catch all exceptions of type `HttpException`, and implement some custom response logic. To do this, we need to add a custom exceptions filter:
```shell
nest g filter common/filters/http-exception
```
This will produce a `http-exception.filter.ts` and its companion test:
```typescript
import { ArgumentsHost, Catch, ExceptionFilter, HttpException } from '@nestjs/common';
import { Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter<T extends HttpException> implements ExceptionFilter {
  catch(exception: T, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();

    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse();
    const error = typeof response === 'string'
      ? { message: exceptionResponse }
      : (exceptionResponse as object);

    response.status(status).json({
      ...error,
      timestamp: new Date().toISOString(),
    });
  }
}
```

We can specify the type of exceptions we want to catch in the `@Catch()` decorator. Then, since we know that only `HttpException`'s can get through, we can also specific that `T extends HttpException`. The `host.switchToHttp()` call returns the native HTTP context that we are in, including the original request and the response that is being produced. The `exception.getResponse()` is the raw exception response, i.e. what would be the response if we didn't do anything.

Now that we have a new filter, we need to apply it in order for it to be used every time an exception is thrown. To do this globally, we can go to `main.ts`:
```typescript
...
import { HttpExceptionFilter } from './common/filters/http-exception.filter';
...
app.useGlobalFilters(new HttpExceptionFilter());
...
```

Filters can also be applied by a specific module by doing:
```typescript
...
import { APP_FILTER } from '@nestjs/core';
import { HttpExceptionFilter } from '../common/filters/http-exception.filter';
...
providers: [
  {
    provide: APP_FILTER,
    useClass: HttpExceptionFilter,
  },
],
...
```

this will still register the filter globally, regardless of what module we're adding the filter to. To apply the filter at a controller level:
```typescript
import { ..., UseFilters } from '@nestjs/common';
import { HttpExceptionFilter } from '../common/filters/http-exception.filter';
...
@UseFilters(HttpExceptionFilter)
@Controller('coffees')
...
```

This way only exceptions produced by this controller will be handled by the specified filter. We can also apply the filter to a specific action:
```typescript
...
@UseFilters(HttpExceptionFilter)
@Get()
findAll(@Query() paginationQuery: PaginationQueryDto) {
  ...
}
...
```

## Guards

Guards are ways to check requests and decide which is allowed to be further processed, and which needs to be refused. This is useful for example for authorization and authentication. To create a new guard:
```shell
nest g guard common/guards/api-key
```
```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class ApiKeyGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}
```

To bind the guard globally in `main.ts`:
```typescript
...
import { ApiKeyGuard } from './common/guards/api-key.guard';
...
app.useGlobalGuards(new ApiKeyGuard());
...
```

When a guard returns `false`, NestJS will send to the user agent a response with status `403: Forbidden`. Let's say we want to check an API key:
```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Observable } from 'rxjs';
import { Request } from 'express';

@Injectable()
export class ApiKeyGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest<Request>();
    const authHeader = request.header('Authorization');
    return authHeader === process.env.API_KEY;
  }
}
```

We can attach metadata to route handlers with the `@SetMetadata()` decorator. This takes a key and a value that can be attached to a route. To avoid having to manually repeat the key each time, we can build a custom decorator, like `common/decorators/public.decorator.ts`:
```typescript
improt { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';

export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

Now we can use this decorator inside our controller:
```typescript
...
import { Public } from '../common/decorators/public.decorator';
...
@Public()
@Get()
findAll(@Query() paginationQuery: PaginationQueryDto) {
  ...
}
```

Now let's edit our guard in order to check if the accessed route is public or not:
```typescript
...
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../decorators/public.decorator';
...
constructor(private readonly reflector: Reflector) {}

canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
  const isPublic = this.reflector.get(IS_PUBLIC_KEY, context.getHandler());
  if (isPublic) {
    return true;
  }
  ...
}
```

In this case we call `context.getHandler()` because we want to refer to the route handler, which is the action method we applied the `@Public()` decorator to. Had we wanted to get the class, we'd have used `context.getClass()` instead.

To inject the reflector in the guard, we need to define the guard in a module instead of `main.ts`:
```typescript
import { ApiKeyGuard } from './common/guards/api-key.guard';
impory { APP_GUARD } from '@nestjs/core';
...
providers: [
  {
    provide: APP_GUARD,
    useClass: ApiKeyGuard,
  },
],
...
```

Guards can be applied to controllers and actions as well.

## Interceptors

Interceptors add additional behavior to route handling, without modifying the original actions, and are particularly useful for implementing cross-cutting concerns. Let's create a new interceptor:
```shell
nest g interceptor common/interceptors/wrap-response
```
```typescript
import { CallHandler, ExecutionContext, Injectable, NestInterceptor } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class WrapResponseInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle();
  }
}
```

This method is called during the handling of the request. However, we can also add logic after the handling of the response, because we're using observable, and so we can register some callbacks to be called after the handling of the route is completed: `next.handle().pipe(tap(data => console.log('After...', data)))`.

Again we can register simple interceptors easily in `main.ts`:
```typescript
...
import { WrapResponseInterceptor } from './common/interceptors/wrap-response.interceptor';
...
app.useGlobalInterceptors(new WrapResponseInterceptor());
...
```

We want to wrap all responses, so we can do:
```typescript
...
import { map } from 'rxjs/operators';
...
return next.handle().pipe(map((data) => ({ data })));
...
```

This way we map every response into an object.

Let's say we want to terminate a request after a specific timeout:
```typescript
...
import { timeout, catchError } from 'rxjs/operators';
import { ..., TimeoutError, throwError } from 'rxjs';
import { ..., RequestTimeoutException } from '@nestjs/common';
...
return next.handle().pipe(timeout(3000), catchError(err => {
  if (err instanceof TimeoutError) {
    return throwError(new RequestTimeoutException());
  }
  return throwError(err);
}));
...
```

Interceptors can be registered at controller level and action level as well.

## Pipes

Pipes are used to transform or validate input data, and thus are called just before a method is called. Let's create a pipe:
```shell
nest g pipe common/pipes/parse-int
```
```typescript
import { ArgumentMetadata, Injectable, PipeTransform } from '@nestjs/common';

@Injectable()
export class ParseIntPipe implements PipeTransform {
  transform(value: string, metadata: ArgumentMetadata) {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException(`Validation failed. "${value}" is not an integer.);
    }
    return val;
  }
}
```

Pipes are added globally to `main.ts` as usual:
```typescript
...
import { ParseIntPipe } from './common/pipes/parse-int.pipe';
...
app.useGlobalPipes(new ParseIntPipe());
...
```

In addition to be applicable to modules, controllers, and actions, pipes can also be applied to action arguments:
```typescript
...
import { ParseIntPipe } from './common/pipes/parse-int.pipe';
...
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  ...
}
...
```

this way the pipe will be applied only to that specific argument.

## Middleware

Middleware functions are called during the processing of a request, before the route handler, and any other runtime handling component like pipes, guards and interceptors. Middlewares have access to the request and response, and are not tied to any specific method, but rather to a specified path. Since middlewares are included in the request-response cycle, each of them can either terminate the cycle, by returning some kind of response, or call the next middleware function.

To generate a middleware class:
```shell
nest g middleware common/middleware/logging
```
```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';

@Injectable()
export class LoggingMiddleware implements NestMiddleware {
  use(req: any, res: any, next: () => void) {
    console.time('Request-response time');
    res.on('finish', () => console.timeEnd('request-response time'));
    next();
  }
}
```

To register the middleware, the module needs to implement `NestModule`:
```typescript
...
import { ..., NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggingMiddleware } from '../common/middleware/logging.middleware';
...
export class CommonModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggingMiddleware).forRoutes('*');
  }
}
...
```

Here we're specified that this middleware should be applied to all routes `*`. Alternatively, we could've used `coffees`, or `{ path: 'coffees', method: RequestMethod.GET }`. We could exclude some routes with `consumer.apply(LoggingMiddleware).exclude('coffee').forRoutes('*');`.

## Custom param decorators

To create a custom decorator let's create a `common/decorators/protocol.decorator.ts`:
```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const Protocol = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.protocol;
  },
);
```

To use this decorator in a controller:
```typescript
...
import { Protocol } from '../common/decorators/protocol.decorator';
...
@Get()
async findAll(
  @Protocol('https') protocol: string,
  @Query() paginationQuery: PaginationQueryDto,
) {
  ...
}
...
```

The value passed to the decorator will be available in the `data` argument of the decorator definition.

## Testing

NestJS provides built-in integration with the Jest testing framework:
```shell
npm run test
npm run test:cov
npm run test:e2e
```

Unit tests are located in the `spec.ts` files, and they are related to specific controllers, services, etc. End-to-end tests are instead located in a `test` folder, sibling of `src`, and are inside `e2e-spec.ts` files; they are related to specific features of functionality of the application.

The `coffee.service.spec.ts` file is the following:
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { CoffeeService } from './coffees.service';

describe('CoffessService', () => {
  let service: CoffeesService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [CoffeesService],
    }).compile();

    service = module.get<CoffeesService>(CoffeesService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });
});
```

Before each test is run, we need to instantiate a module, in order to provide the `CoffeesService` we want to test. The `Test` class is useful to mock the full NestJS runtime, for example by providing hooks, mocks, etc. If we need to retrieve request scope, or transient scope providers, we need to call `service = module.resolve<CoffeesService>(CoffeesService)` instead.

To run this test:
```shell
npm run test:watch -- coffees.service
```

The `test:watch` command runs a test in watch mode, i.e. automatically re-running the test every time any file is changed.

Our default test definition is lacking several dependencies needed to test the `CoffeesService`:
```typescript
...
import { Connection } from 'typeorm';
import { getRepositoryToken } from '@nestjs/typeorm';
import { Flavor } from './entities/flavor.entity';
import { Coffee } from './entities/coffee.entity';
...
const module: TestingModule = await Test.createTestingModule({
  providers: [
    CoffeesService,
    { provide: Connection, useValue: {} },
    { provide: getRepositoryToken(Flavor), useValue: {} },
    { provide: getRepositoryToken(Coffee), useValue: {} },
  ],
}).compile();
```

Here we're injecting empty objects, just to let the first test work.

Let's test the `findOne()` method:
```typescript
...
import { ..., Repository } from 'typeorm';
...
type MockRepository<T = any> = Partial<Record<keyof Repository<T>, jest.Mock>>;
const createMockRepository = <T = any>(): MockRepository<T> => ({
  findOne: jest.fn(),
  create: jest.fn(),
});
...
let coffeeRepository: MockRepository;
...
{ provide: getRepositoryToken(Flavor), useValue: createMockRepository() },
{ provide: getRepositoryToken(Coffee), useValue: createMockRepository() },
...
coffeeRepository = module.get<MockRepository>(getRepositoryToken(Coffee));
...
describe('findOne', () => {
  describe('when coffee with ID exists', () => {
    it('should return the coffee object', async () => {
      const coffeeId = '1';
      const expectedCoffee = {};

      coffeeRepository.findOne.mockReturnValue(expectedCoffee);
      const coffee = await service.findOne(coffeeId);
      expect(coffee).toEqual(expectedCoffee);
    });
  });
  describe('otherwise', () => {
    it('should throw the "NotFoundException"', async () => {
      const coffeeId = '1';
      coffeeRepository.findOne.mockReturnValue(undefined);

      try {
        await service.findOne(coffeeId);
      } catch (err) {
        expect(err).toBeInstanceOf(NotFoundException);
        expect(err.message).toEqual(`Coffee #${coffeeId} not found');
      }
    });
  });
});
...
```

The default end-to-end test is `app.e2e-spec.ts`:
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from './../src/app.module';

describe('AppController (e2e)', () => {
  let app: INestApplication;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/ (GET)', () => {
    return request(app.getHttpServer())
      .get('/')
      .expect(200)
      .expect('Hello World');
  });
});
```

Unlike unit tests, in end-to-end tests we don't have to create single classes, but the entire application, and we do that with `app = moduleFixture.createNestApplication()`, which is then followed by `await app.init()`. Actual tests simulate HTTP requests by using the `supertest` library. To run end-to-end tests:
```shell
npm run test:e2e
```

Beware that the application built in the end-to-end tests does not run `main.ts`, so any guard, interceptor, etc. defined there won't be activated during the end-to-end tests. It may happen that at the end of a test we get the message: `Jest did not exit one second after the test run has completed.`. This means that there is some operation that has been left running. In this case, it's an open database connection. To fix this:
```typescript
...
afterAll(async () => {
  await app.close();
});
...
```

This will shutdown the application after all tests are completed, calling `onModuleDestroy` and `onApplicationShutdown` lifecycle hooks, which perform all shutdown operations, including closing database connections that are still open. Additionally, it's also better to create the application only once for each test, so we should use `beforeAll` instead of `beforeEach`:
```typescript
...
beforeAll(async() => {
  const moduleFixture: Testing module = await Test.createTestingModule({
    ...
  });
  ...
});
...
```

Each end-to-end test should ideally focus on a single feature, so we need to be able to load only the parts of the application that are needed for that feature, instead of the whole thing for every test. Let's create a `tests/coffee/coffees.e2e-spec.ts` foòe, to contain end-to-end tests for the `coffee` module:
```typescript
import { INestApplication } from '@nestjs/common';
import { TestingModule, Test } from '@nestjs/testing';
import { CoffeesModule } from '../../src/coffees/coffees.module';

describe('[Feature] Coffees - /coffees', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [CoffeesModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it.todo('Create [POST /]');
  it.todo('Get all [GET /]');
  it.todo('Get one [GET /:id]');
  it.todo('Update one [PATCH /:id]');
  it.todo('Delete one [DELETE /:id]');

  afterAll(async () => {
    await app.close();
  });
});
```

This time we're including only the `CoffeesModule` in the application we're building for this test. To run this specific end-to-end test only:
```shell
npm run test:e2e -- coffees
```

Again we are missing the database connection dependency. When it comes to end-to-end tests, it's best to run them against a real database, different than the one actually used by the application. We can add another database in `docker-compose.yml`:
```yaml
...
test-db:
  image: postgres
  restart: always
  ports:
    - "5433:5432"
  environment:
    POSTGRES_PASSWORD: pass123
```

We can now add pre- and post- hooks to our tests in the definitions of NPM scripts:
```json
...
"scripts": {
  ...
  "pretest:e2e": "docker-compose up -d test-db",
  "posttest:e2e": "docker-compose stop test-db && docker-compose rm -f test-db"
},
...
```

This way the container for the test database will be automatically created before each test run, and destroyed afterwards. Lastly, we need to initialize a connection to this test database inside our end-to-end test:
```typescript
...
import { TypeOrmModule } from '@nestjs/typeorm';
...
imports: [
  ...
  TypeOrmModule.forRoot({
    type: 'postgres',
    host: 'localhost',
    port: 5433,
    username: 'postgres',
    password: 'pass123',
    database: 'postgres',
    autoLoadEntities: true,
    synchronize: true,
  }),
]
...
```

If we added some configuration to `main.ts`, we need to replicate it in the end-to-end test, because the application is not built by running `main.ts`:
```typescript
...
import { ValidationPipe } from '@nestjs/common';
...
app = moduleFixture.createNestApplication();
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  transform: true,
  forbidNonWhitelisted: true,
  transformOptions: {
    enableImplicitConversion: true,
  },
}));
...
...
```

Let's add a test for the `create` endpoint:
```typescript
...
import { ..., HttpStatus } from '@nestjs/common';
import * as request from 'supertest';
import { CreateCoffeeDto } from 'src/coffees/dto/create-coffee.dto';
...
const coffee = {
  name: 'Shipwreck Roast',
  brand: 'Buddy Brew',
  flavors: ['chocolate', 'vanilla'],
};
...
it('Create [POST /]', () => {
  return request(app.getHttpServer())
    .post('/coffees')
    .send(coffee as CreateCoffeeDto)
    .expect(HttpStatus.CREATED)
    .then(({ body }) => {
      const expectedCoffee = jasmine.objectContaining({
        ...coffee,
        flavors: jasmine.arrayContaining(
          coffee.flavors.map(name => jasmine.objectContaining({ name })),
        ),
      });
      expect(body).toEqual(expectedCoffee);
    });
});
...
```
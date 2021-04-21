# Projeto em Node da Rocketseat da NLW 05

## Firts Day

## Second Day
1. Instalação do *TypeORM* e *SQLite*:
  ```console
    yarn add typeorm reflect-metadata sqlite3
  ```

2. Criação do arquivo **ormconfig.json** na raíz, com o conteúdo: 
  ```json
    {
      "type": "sqlite"
    } 
  ```

3. Criar um diretório **database** dentro de **src**;

4. Dentro de **database** criar um arquivo **index.ts** que vai criará a conexão com o banco:  
  ```ts
    import { createConnection } from 'typeorm';

    createConnection();
  ```

5. Importar a conexão com o banco em **server.ts** 
  ```ts
    import './database'
  ```
6. Criar diretório **migration** dentro de **database**

7. Utiliza o typeorm para ajudar a criar os arquivos de *migrations*, para isso temos que efetuar mais algumas configurações no arquivo **ormconfig.json**:
  ```json
    {
      "type": "sqlite",
      "database": "./src/database/database.sqlite",
      "migrations": [
        "./src/database/migrations/*.ts"
      ],
      "cli": {
        "migrationsDir": "./src/database/migrations"
      }
    }
  ```

8. Criar um script no **package.json** para facilitar a criação das *migrations*:
  ```json
    "scripts": {
      "dev:server": "ts-node-dev src/server.ts",
      "typeorm": "ts-node-dev node_modules/typeorm/cli.js"
    },  
  ```

9. Criar de fato as *migrations*:
  ```console
    yarn typeorm migration:create -n CreateSettings
  ```

10. Preencher método **up** da migration *CreateSettings* com as informações da tabela a ser criada:
  ```ts
    await queryRunner.createTable(
        new Table({
            name: 'settings',
            columns: [
                {
                    name: "id",
                    type: "uuid",
                    isPrimary: true
                },
                {
                    name: "username",
                    type: "varchar"
                },
                {
                    name: 'chat',
                    type: 'boolean',
                    default: true
                },
                {
                    name: 'create_at',
                    type: 'timestamp',
                    default: 'now()'
                },
                {
                    name: 'update_at',
                    type: 'timestamp',
                    default: 'now()'
                }
            ]
        })
    )
  ```

11. Preencher método **down** da migration *CreateSettings* com a exclusão da tabela que foi criada no método **up**:
  ```ts
    await queryRunner.dropTable('settings')
  ```

12. Executar a migration para que a tabela seja criada efetivamente da base de dados: 
  ```console
    yarn typeorm migration:run
  ```

13. Criar diretório **entities** dentro de **src**

14. Dentro de **entities** criar arquivo **Setting.ts** que será nossa entidade relacionada a tabela que acabamos de criar.

15. O arquivo **Settings.ts** ficará da seguinte forma: 
  ```ts
    import { Entity, Column, CreateDateColumn, UpdateDateColumn, PrimaryColumn } from "typeorm"

    @Entity('settings')
    class Setting {

      @PrimaryColumn()
      id: string;

      @Column()
      username: string;

      @Column()
      chat: boolean;

      @UpdateDateColumn()
      update_at: Date;

      @CreateDateColumn()
      create_at: Date;
    }

    export { Setting }
  ```

16. Para que as *annotations* funcionem é preciso descomentar as seguintes linhas no arquivo **tsconfig.json**: 
  ```json
    "experimentalDecorators": true, 
    "emitDecoratorMetadata": true, 
  ```

  17. Agora vamos delegar a responsabilidade da geração do *id* para nossa aplicação, não deixaremos essa responsabilidade para o banco, pois o mesmo pode variar de banco para banco. Para isso vamos iniciar instalando a biblioteca **uuid**:
    ```console
      yarn add uuid
      yarn add @types/uuid -D
    ```
  
  18. De volta ao arquivo **Setting.ts** vamos fazer a importação da biblioteca **uuid** e criar um construtor para que seja atribuído um código *uuid* a propriedade *id*: 
    ```ts
      import { v4 as uuid } from 'uuid';

      constructor() {
        if (!this.id) {
          this.id = uuid();
        }
      }

    ``` 

  19. Agora temos que informar ao *typeorm* onde estão nossa entidades, para isso temos que adicionar as seguintes linhas no arquivo **ormconfig.json**: 
    ```json
      "entities": ["./src/entities/*.ts"]
    ```

  20. Vamos começar a criar nossos repositórios, para primeiro vamos criar um diretório **repositories**, dentro de **src**, que irá conter todos os nossos repositórios.

  21. Dentro dele vamos criar um arquivo chamado **SettingsRepository.ts**, com o seguinte conteúdo: 
    ```ts
      import { EntityRepository, Repository } from 'typeorm'
      import { Setting } from '../entities/Setting';

      @EntityRepository(Setting)
      class SettingsRepository extends Repository<Setting> {}

      export default SettingsRepository;
    ```

  22. Agora vamos criar nosso arquivo de rotas dentro da pasta **src** com o nome de **routes.ts** e que irá conter a nossa primeira rota: 
    ```ts
      import { Router } from 'express';
      import { getCustomRepository } from 'typeorm';
      import SettingsRepository from './repositories/SettingsRepository';

      const routes = Router();

      routes.post('/settings', async (request, response) => {
        const { chat, username } = request.body;

        const settingsRepository = getCustomRepository(SettingsRepository);

        const settings = settingsRepository.create({
          chat,
          username
        })

        await settingsRepository.save(settings);

        return response.json(settings);
      })

      export default routes; 
    ```
  
  23. Após a criação do arquivo de rotas, vamos apagar a rota que criamos para teste no arquivo **server.ts** e vamos importar o arquivo de rotas, além disso vamos dizer ao express que as requisições virão em formato JSON, o arquivo **server.ts** ficará assim:
    ```ts
      import express from 'express';
      import './database';
      import routes from './routes';

      const app = express();

      app.use(express.json());
      app.use(routes);

      app.listen(3333, () => console.log('Server is running on port 3333'))   
    ```

  24. Por fim vamos refatorar um pouco nosso código criando uma camada de controller, para isso vamos criar um diretório **controllers** dentro de **src** e dentro dele criar um arquivo **SettingsController.ts** e vamos transportar todo código da nossa rota */settings* para esse novo controller, ficando assim:
    *SettingsController.ts*: 
    ```ts
      import { Request, Response } from 'express'
      import { getCustomRepository } from 'typeorm';
      import SettingsRepository from '../repositories/SettingsRepository';

      class SettingsController {

        async create(request: Request, response: Response) {
          const { chat, username } = request.body;

          const settingsRepository = getCustomRepository(SettingsRepository);

          const settings = settingsRepository.create({
            chat,
            username
          })

          await settingsRepository.save(settings);

          return response.json(settings);
        }
      }

      export default SettingsController;  
    ```

    *routes.ts*:
    ```ts
      import { Router } from 'express';
      import SettingsController from './controllers/SettingsController';

      const routes = Router();
      const settingsController = new SettingsController();

      routes.post('/settings', settingsController.create)

      export default routes; 
    ``` 
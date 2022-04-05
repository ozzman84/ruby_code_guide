### 1. Create Rails Application
  * From the command line, start a new rails app.  For example:  
    ```zsh
    $ rails _5.2.6_ new project_name -T -d postgresql
    ```
  * API only application
    ```zsh
    $ rails _5.2.6_ new project_name -T -d postgresql --api
    ```

  * Install Rspec w/helper files

    ```zsh
    $ rails g rspec:install
    ```

  * SimpleCov - Add to top of `spec/rails_helper.rb`
    ```ruby
    require 'simplecov'
    SimpleCov.start
    ```

  * Shoulda-matchers - Add to the bottom of `spec/rails_helper.rb`
    ```ruby
    Shoulda::Matchers.configure do |config|
      config.integrate do |with|
        with.test_framework :rspec
        with.library :rails
      end
    end
    ```

### 3. Create Database and Resources
  * To drop, create, migrate, or seed the database in the terminal run:
      ```zsh
    $ rails db:{drop,create}
    ```

  * Model Generator:
    ```zsh
    $ rails g model User title body:text
    ```

  * Child:
    ```zsh
    $ rails g model Comment author_name:string body:integer user:references
    ```

  * After a migration is successfully generated, run `$ rails db:migrate` in the terminal.

  * Add relationship tests:

    ```ruby
   describe "relationships" do
      it { should belong_to(:user) }
      it { should have_many(:comments) }
      it { should have_many(:users).through(:taggings) }
   end
    ```

  * Add model relationships:  

    ```ruby
    # app/models/comment.rb
     belongs_to :article   
     has_many :comments
     has_many :articles, through: :taggings
    ```

  * Validation Tests
    ```ruby
   describe 'validations' do
     it { should validate_presence_of(:title) }
     it { should validate_presence_of(:body) }
   end
    ```

  * Model Validations  

    ```ruby
      validates :title, :body, presence: true
    ```
  * Controller Generator:

    ```zsh
    $ rails g controller api/v1/Users index create update --skip-routes --no-test-framework --skip-helper --skip-template-engine
    ```

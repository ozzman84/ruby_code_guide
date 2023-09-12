# Rails Code Guide

Updated by: **Ozzie Osmonson**

[![LinkedIn: ozzman84][linkedin-badge]][LinkedIn]
[![Email: mikeosmonson@gmail.com@gmail.com][gmail-badge]][gmail]
[![GitHub: ozzman84][github-follow-badge]][GitHub]

## Table of Contents

- [Getting Started](#getting-started)
  1. [Create Rails Application](#1-create-rails-application)
  2. [Environment Setup](#2-environment-setup)
  3. [Create Database and Resources](#3-create-database-and-resources)
- [Models and Relationships](#models-and-relationships)
  - [Relationship Summary](#relationship-summary)
  - [Relationship Examples](#relationship-examples)
    - [One-to-Many](#one-to-many)
    - [Many-to-Many](#many-to-many)
- [Updating Database Schema](#updating-database-schema)
  - [Edit Table Column Data Type](#edit-table-column-data-type)
  - [Add New Column to Table](#add-new-column-to-table)
- [Testing](#testing)
- [Create Routes](#create-routes)
- [Create Controllers](#create-controllers)
- [Create Views](#create-views)
- [Rails Helpers](#rails-helpers)
  - [form_with](#form_with)
  - [validates](#validates)
  - [link_to](#link_to)

---

## Getting Started

### 1. Create Rails Application
  * From the command line, start a new rails app.  For example:  
    ```zsh
    rails _7.0.6_ new project_name -T -d postgresql -c=tailwind
    ```
  * API only application
    ```zsh
    rails _7.0.6_ new project_name -T -d postgresql --api
    ```


### 2. Environment Setup
  * Gem List

    ```ruby
    # Gemfile
    group :development, :test do
      gem 'pry'
      gem 'rspec-rails'
      gem 'capybara'
      gem 'launchy'
      gem 'simplecov'
      gem 'shoulda-matchers'
      gem 'orderly'
      gem 'factory_bot_rails'
      gem 'faker'
    end
    ```

  * Bundle Install

    ```zsh
    bundle install
    ```

  * Install Rspec w/helper files

    ```zsh
    rails g rspec:install
    ```


  * SimpleCov - Add to `spec/rails_helper.rb` (reference: [simplecov docs](https://github.com/simplecov-ruby/simplecov)):

    ```ruby
    # spec/rails_helper.rb
    require 'simplecov'
    SimpleCov.start
    ```

  * Factory_bot methods:

  ```ruby
  # spec/rails_helper.rb
  config.include FactoryBot::Syntax::Methods
  ```

  * Shoulda-matchers - Add to the bottom of `spec/rails_helper.rb` (reference: [shoulda-matchers docs](https://github.com/thoughtbot/shoulda-matchers)):

    ```ruby
    # spec/rails_helper.rb
    Shoulda::Matchers.configure do |config|
      config.integrate do |with|
        with.test_framework :rspec
        with.library :rails
      end
    end
    ```

  * From the command line run `$ bundle install`. Sometimes it will be necessary to run `$ bundle update` to get the latest versions of the gems.


### 3. Create Database and Resources
  * To drop, create, migrate, or seed the database in the terminal run:
    ```zsh
    rails db:{drop,create,migrate,seed}
    ```

  * Rails Generators ([Rails Guides: AR Migrations](https://guides.rubyonrails.org/v5.2/active_record_migrations.html)) in the terminal run:

      * Model Generator:
        ```zsh
        rails g model User title:string body:text
        ```

      * Child:
        ```zsh
        rails g model Comment author_name:string body:integer user:references
        ```

      * Add a relationship post migration:
        ```zsh
        rails g migration AddArticlesToComments article:references
        ```

        ```ruby
        # db/migrate/[timestamp]_add_articles_to_comments.rb
        def change
                       # primary table      # adds foreign key to primary table
         add_reference :comments, :article, foreign_key: true
                                  # foreign table
        end
        ```

      * Migration file (`db/migrate/` directory):
        ```ruby
        # db/migrate/[timestamp]_create_articles.rb
        class CreateUser < ActiveRecord::Migration[5.2]
          def change
            create_table :articles do |t|
              t.string :title
              t.text :body

              t.timestamps
            end
          end
        end
        ```
  * After a migration is successfully generated, run `$ rails db:migrate` in the terminal.

---

## Models and Relationships
Reference: [Rails Guides: AR Associations](https://guides.rubyonrails.org/v5.2/association_basics.html)


### Relationships

  * Add relationship tests:

    ```ruby
    # spec/models/comment_spec.rb
    require "rails_helper"

    describe Comment, type: :model do
       describe "relationships" do
          it { should belong_to(:user) }
          it { should have_many(:comments) }
          it { should have_and_belong_to_many(:posts) }
          it { should have_many(:users).through(:taggings) }
          it { should have_one(:profile).dependent(:destroy) }
          it { should have_many(:orders).order('orders.fulfilled_at DESC') }
          it { should have_many(:receipts).through(:orders) }
          it { should have_many(:active_receipts).conditions('receipts.active = 1') }
          it { should belong_to(:category).class_name('MenuCategory') }
          it { should belong_to(:category).class_name('MenuCategory') }
          it {should accept_nested_attributes_for(:user) }
       end
    end
    ```

  * Add model relationships:  

    ```ruby
    # app/models/comment.rb
    class Comment < ApplicationRecord  
       belongs_to :article   
       has_many :comments
       has_many :articles, through: :taggings
    end
    ```

#### Validations
References: [Rails Guides: Validations](https://guides.rubyonrails.org/v5.2/active_record_validations.html), [Shoulda-Matchers Docs](https://github.com/thoughtbot/shoulda-matchers#matchers)

  * Validation Tests
    ```ruby
    # spec/models/comment_spec.rb
    require "rails_helper"

    RSpec.describe Comment, type: :model do
      describe 'validations' do
        before { User.create(email: 'john@example.com') }
        it { should_not have_db_column(:admin).of_type(:boolean)
        it { should have_db_column(:salary).
          of_type(:decimal).
          with_options(:precision => 10, :scale => 2) }
        it { should have_readonly_attributes(:password) }
        it { should have_db_index(:id) }
        it { should validate_presence_of(:title) }
        it { should validate_presence_of(:name).
          with_message(/is not optional/) }
        it { should validate_uniqueness_of(:email) }
        it { should validate_uniqueness_of(:keyword).with_message(/dup/) }
        it { should validate_uniqueness_of(:email).scoped_to(:name) }
        it { should validate_uniqueness_of(:email).
          scoped_to(:first_name, :last_name) }
        it { should validate_uniqueness_of(:keyword).case_insensitive }
        it { should validate_length_of(:operating_state).is_equal_to(2) }
        it { should validate_length_of(:name).
              is_at_least(3).
              with_short_message(/not long enough/) }
        it { should validate_length_of(:ssn).
              is_equal_to(9).
              with_message(/is invalid/) }
        it { should validate_acceptance_of(:terms) }
        it { should validate_format_of(:ssn).with(/\A\d{3}-\d{2}-\d{4}\Z/) }
        it { should validate_format_of(:name).
          with('12345').
          with_message(/is not optional/) }
        it { should validate_format_of(:name).
          not_with('12D45').
          with_message(/is not optional/) }
        it { should validate_numericality_of(:age).is_greater_than(0).is_less_than(150) }
        it { should ensure_inclusion_of(:age).in_range(0..100) }
        it { should_not allow_mass_assignment_of(:password) }
        it { should allow_mass_assignment_of(:first_name) }
        it { should validate_format_of(:first_name).with("Carl") }
        it { should validate_confirmation_of(:password) }
      end
    end
    ```

  * Model Validations  

    ```ruby
    # app/models/tagging.rb
    class Tagging < ApplicationRecord
      validates :title, :body, presence: true
      validates :name, presence: true, uniqueness: true, on: [:create, :update]
      validates :name, presence: true, on: :admin_area
      validates :street_address, presence: true
      validates :state, presence: true, { is: 2 }
      validates :holiday,  uniqueness: { scope: :year, message: 'yearly only' }
      validates :terms,    acceptance: true
      # Unique
      validates :slug,     uniqueness: true
      validates :slug,     uniqueness: { case_sensitive: false }
      validates :holiday,  uniqueness: { scope: :year, message: 'yearly only' }

      # Format
      validates :code,     format: /regex/
      validates :code,     format: { with: /regex/ }
      # Length
      validates :name,     length: { minimum: 2 }
      validates :bio,      length: { maximum: 500 }
      validates :password, length: { in: => 6..20 }
      validates :number,   length: { is: => 6 }
      # Include/exclude
      validates :gender,   inclusion: %w(male female)
      validates :gender,   inclusion: { in: %w(male female) }
      validates :lol,      exclusion: %w(xyz)
      # Numeric
      validates :points,   numericality: true
      validates :played,   numericality: { only_integer: true }
      # ... greater_than, greater_than_or_equal_to,
      # ... less_than, less_than_or_equal_to
      # ... odd, even, equal_to

      # Validate the associated records to ensure they're valid as well
      has_many :books
      validates_associated :books

      # Length (full options)
      validates :content, length: {
        minimum:   300,
        maximum:   400,
        tokenizer: lambda { |str| str.scan(/\w+/) },
        too_short: "must have at least %{count} words",
        too_long:  "must have at most %{count} words" }
        # Conditional
        validates :description, presence: true, if: :published?
        validates :description, presence: true, if: lambda { |obj| .. }
    end
    ```

---

* Controller Generator:

  ```zsh
  rails g controller api/v1/Users index create update --skip-routes --no-test-framework --skip-helper --skip-template-engine
  ```

* Flags:

  ```zsh
  $ --skip-routes
  $ --no-test-framework
  $ --skip-helper
  $ --skip-template-engine
  ```

  ```ruby
  # app/controllers/api/v1/users_controller.rb
  class Api::V1::UsersController < ApplicationController
    def index

    end

    def create

    end

    def update

    end
  end
  ```
  ---

## Updating Database Schema

### Edit Table Column Data Type
Reference: [Rails Guides: Changing Columns](https://guides.rubyonrails.org/v5.2/active_record_migrations.html#changing-columns)

When you need to make an edit **after** you have migrated, you should create a **new** migration.

1. Generate a new migration in the terminal.  
   For example, to change the datatype of the `body` column to `text` in the `Comments` table:
   ```zsh
   # Example Structure:

   $ rails generate migration ChangeColumnNameToBeDatatypeInTableName
   ```
   ```zsh
   # Practical Example:

   $ rails generate migration ChangeBodyToBeTextInComments
   ```

2. Open the migration file and put in the change.
   For example:
   ```ruby
   # Example Structure:

   # db/migrate/[timestamp]_change_column_name_to_be_datatype_in_table_name.rb
   class ChangeColumnNameToBeDatatypeInTableName < ActiveRecord::Migration[5.2]
     def change
       change_column :table_name, :column_name, :datatype # This is what you put within the 'change' method code block
     end
   end
   ```

   ```ruby
   # Practical Example:

   # db/migrate/[timestamp]_change_body_to_be_text_in_comments.rb
   class ChangeBodyToBeTextInComments < ActiveRecord::Migration[5.2]
     def change
       change_column :comments, :body, :text # This is what you put within the 'change' method code block
     end
   end
   ```
3. Run `rails db:migrate`.

4. Check database schema file (`db/schema.rb`) to confirm the column data type has been updated.  

### Add New Column to Table
Reference: [Rails Guides: Add Column](https://guides.rubyonrails.org/v5.2/active_record_migrations.html#creating-a-standalone-migration)

1. Generate a new migration in the terminal.  
   For example, to add an `email` column to the `Comments` table:
   ```zsh
   # Example Structure:

   rails g migration AddColumnNameToTableName
   ```

   ```zsh
   # Practical Example:

   rails g migration AddEmailToComments
   ```

2. Open the migration file and put in the change (or to confirm it was automatically generated, if the migration name follows the form `AddXXXToYYY`).
   For example:
   ```ruby
   # Example Structure:

   # db/migrate/[timestamp]_add_column_name_to_table_name.rb
   class AddColumnNameToTableName < ActiveRecord::Migration[5.2]
     def change
       add_column :table_name, :column_name, :datatype # This will be automatically generated if migration name follows the form: "AddXXXToYYY"
                                                       # Otherwise, you can manually input.
     end
   end
   ```

   ```ruby
   # Practical Example:

   # db/migrate/[timestamp]_add_email_to_comments.rb
   class AddEmailToComments < ActiveRecord::Migration[5.2]
     def change
       add_column :comments, :email, :string # This will be automatically generated if migration name follows the form: "AddXXXToYYY"
                                             # Otherwise, you can manually input.      
     end
   end
   ```

3. Run `rails db:migrate`.

4. Check database schema file (`db/schema.rb`) to confirm the new column was created.   

---

## Testing
References:[Rails Guides: AR Validations](https://guides.rubyonrails.org/v5.2/active_record_validations.html), [Shoulda-Matchers Docs](https://github.com/thoughtbot/shoulda-matchers#matchers)

  * Create both a `models` and `features` sub-directory in the `spec` folder (e.g. `spec/models/` and `spec/features/`).
    * In general, `model` tests will test the relationships, validations, and logic of the model methods.
    * In general, `feature` tests will test the display and functionality of the views.

  * Framework for a `feature` test:
    ```ruby
    # Practical Example:

    # spec/features/articles/index_spec.rb
    require 'rails_helper'

    RSpec.describe '/articles/index.html.erb', type: :feature do
      let!(article1) { Article.create!(title: 'Title 1', body: 'Body 1') }
      let!(article2) { Article.create!(title: 'Title 2', body: 'Body 2') }

      describe 'as a user' do
        describe 'when I visit the articles index' do
          it 'displays all the articles' do
            visit '/articles'

            expect(page).to have_content(article1.title)
            expect(page).to have_content(article2.title)
          end
        end
      end
    end
    ```

---

## Create Routes

  * Add code into `config/routes.rb`, such as one of the following examples:
    ```ruby
    # config/routes.rb
    Rails.application.routes.draw do
      root to: 'welcome#index'

      get '/articles/:id', to: 'articles#show'

      namespace :api do
        namespace :v1 do
          resources :managers # Example Rails resources all RESTful routes
          resources :users, only: [:index, :show]
          resources :articles, except: [:destroy, :index]
        end
      end
    end
    ```

  * Seven RESTful routes (actions): `index`, `new`, `create`, `show`, `edit`, `update`, and `destroy`

    Route                                            | HTTP Verb | URI             | Controller Action | Description
    ------------------------------------------------ | ------ | ------------------ | ------- | -----------
    `get '/articles', to: "articles#index`           | GET    | /articles          | index   | Display a list of all `articles`
    `get '/articles/new', to: "articles#new`         | GET    | /articles/new      | new     | Show form to make a new `article`
    `post '/articles', to: "articles#create"`        | POST   | /articles          | create  | Add new `article` to the database, then redirect
    `get '/articles/:id', to: "articles#show"`       | GET    | /articles/:id      | show    | Show information about a particular `article`
    `get '/articles/:id/edit', to: "articles#edit"`  | GET    | /articles/:id/edit | edit    | Show from to edit an existing `article`
    `patch '/articles/:id', to: "articles#update"`   | PATCH  | /articles/:id      | update  | Update an existing `article`, then redirect
    `delete '/articles/:id', to: "articles#destroy"` | DELETE | /articles/:id      | destroy | Delete a particular `article`, then redirect

  * To display all routes in the rails application run:
    ```zsh
    rails routes
    ```

  * To display all routes for a specific controller in the rails application, run:
    ```zsh
    rails routes -c resource_name
    ```

    ```zsh
    rails routes -c articles
    ```

---

## References

  * [Rails Guides v5.2](https://guides.rubyonrails.org/v5.2/)
  * Style Guides:
    * [The Rails Style Guide](https://rails.rubystyle.guide/)
    * [The RSPec Style Guide](https://rspec.rubystyle.guide/)
    * [The Ruby Syle Guide](https://rubystyle.guide/)


<!-- LINKS AND BADGES -->
[GitHub]: https://github.com/ozzman84
[gmail]: mailto:mikeosmonson@gmail.com
[LinkedIn]: https://www.linkedin.com/in/ozzman84/

[github-follow-badge]: https://img.shields.io/github/followers/ozzman84?label=follow&style=social
[gmail-badge]: https://img.shields.io/badge/gmail-ozzman84@gmail.com-green?style=flat&logo=gmail&logoColor=white&color=white&labelColor=EA4335
[linkedin-badge]: https://img.shields.io/badge/Ozzie--Osmonson-%23OpenToWork-green?style=flat&logo=Linkedin&logoColor=white&color=success&labelColor=0A66C2

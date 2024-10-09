# FormWizard

FormWizard is a Ruby gem that provides a simple and flexible way to build multi-step forms (wizards) in Ruby on Rails applications. It leverages ActiveModel to handle validations and attribute management, making it easy to integrate with your existing models and controllers.

## Features

- **Multi-step forms:** Easily define forms with multiple steps.
- **Attribute management:** Define attributes for each step with default values.
- **Validations:** Add validations specific to each step.
- **Model synchronization:** Automatically sync form attributes with your models.
- **Session handling:** Persist form data across requests using the session.
- **Partial rendering:** Render views based on the current step.

## Example

This is a working example of a multi-step onboarding form. It uses the `step` method to define each step of the wizard.

#### Form object

```ruby
# app/forms/onboarding_form.rb
class OnboardingForm < FormWizard::Form
  step :terms_and_conditions do
    attribute :toc_accepted
    validates :toc_accepted, acceptance: true
  end

  step :profile do
    attribute :name, on: :profile
    validates :name, presence: true, length: { minimum: 2 }

    attribute :account_name, on: :account, column: :name
    validates :account_name, presence: true, length: { minimum: 5 }, allow_blank: true

    attribute :locale, on: :user
    attribute :time_zone, on: :user
  end

  def persist
    self.account_name = name if account_name.blank?

    ActiveRecord::Base.transaction do
      user.save!
      Member.create!(user: user, account: account, owner: true)
    end
  end
end
```

## Installation

Add this line to your application's `Gemfile`:

```ruby
gem "form_wizard"
```

and then execute:

```bash
bundle install
```

## Model synchronization

Pass a model object to the form

```ruby
# Pass an instance of Profile to the form object:
@profile = current_user.profile
@form = OnboardingForm.new(models: { profile: @profile })
```

This allows us to synchronize form attributes with your models using the `:on` option when defining form attributes:

```ruby
step :profile do
  attribute :name, on: :profile
end
```

FormWizard handles synchronization with the models attributes automatically. You can access the updated object inside the `persist` method:

```ruby
def persist
  profile.save!
end
```

## Validations

Validations can be added within each step using the validates method. These validations are conditional and only run on the current step.

```ruby
step :terms do
  validates :toc_accepted, acceptance: true
end
```

# RSpec Rails Tester

A specialized agent for writing RSpec tests in Rails applications following a specific, opinionated style.

## Core Testing Philosophy

- **Request specs over controller tests**: Always write request specs, never controller tests
- **Setup outside `it` blocks**: All data creation and scenario setup happens in `let`, `let!`, or `subject` blocks
- **Concise assertions**: The `it` block should only contain `subject` (if needed) and the expectation
- **Context-driven organization**: Use `context` blocks to describe scenarios, typically starting with "when..."
- **DRY principles**: Avoid repetition; extract common setup to shared contexts or higher-level `let` blocks
- **Smart factory usage**: Prefer `build` and `build_stubbed` over `create` when appropriate (when persistence isn't required)

## Test Structure Pattern

### Good Pattern

```ruby
context "when condition exists" do
  let!(:resource) { create(:resource, attribute: value) }
  let(:params) { { key: "value" } }

  subject { post resource_path, params: params }

  it "performs the expected action" do
    expect { subject }.to change(Model, :count).by(1)
  end

  it "has the correct attributes" do
    subject
    expect(Model.last).to have_attributes(key: "value")
  end
end
```

### Bad Pattern (Never Do This)

```ruby
it "does something" do
  resource = create(:resource)  # NO - create should be in let!
  post resource_path            # NO - request should be in subject
  expect(response).to be_successful
end
```

## Key Rules

1. **HTTP requests go in `subject` blocks**
   ```ruby
   subject { post invitations_path, params: { invitation: invitation_params } }
   ```

2. **Data setup uses `let` or `let!`**
   - Use `let!` when the record must exist before the action
   - Use `let` for lazy evaluation
   - Use `build`/`build_stubbed` when persistence isn't needed

3. **Context blocks describe scenarios**
   ```ruby
   context "when user is unauthorized" do
   context "when email is duplicate" do
   context "with invalid params" do
   ```

4. **`it` blocks are concise**
   ```ruby
   it "creates the record" do
     expect { subject }.to change(Model, :count).by(1)
   end

   it "returns success status" do
     subject
     expect(response).to have_http_status(:ok)
   end
   ```

5. **Call `subject` explicitly when needed**
   - Use `expect { subject }` for testing changes
   - Call `subject` before assertions that check the result state

## Common Scenarios

### Testing Record Creation

```ruby
context "with valid params" do
  let(:params) { { title: "Test", description: "Description" } }
  subject { post items_path, params: { item: params } }

  it "creates the item" do
    expect { subject }.to change(Item, :count).by(1)
  end

  it "has correct attributes" do
    subject
    expect(Item.last).to have_attributes(params)
  end

  it "returns created status" do
    subject
    expect(response).to have_http_status(:created)
  end
end
```

### Testing Authorization

```ruby
context "when accessing another organization's resource" do
  let!(:other_org) { create(:organization) }
  let!(:other_resource) { create(:resource, organization: other_org) }

  subject { get resource_path(other_resource) }

  it "returns not found" do
    subject
    expect(response).to have_http_status(:not_found)
  end
end
```

### Testing Duplicate Prevention

```ruby
context "when email already exists" do
  let!(:existing_record) { create(:invitation, email: "test@example.com") }
  let(:params) { { email: "test@example.com" } }

  subject { post invitations_path, params: { invitation: params } }

  it "does not create a duplicate" do
    expect { subject }.not_to change(Invitation, :count)
  end

  it "returns unprocessable entity" do
    subject
    expect(response).to have_http_status(:unprocessable_entity)
  end
end
```

### Testing Updates

```ruby
context "updating attributes" do
  let!(:resource) { create(:resource, title: "Old Title") }
  let(:new_params) { { title: "New Title" } }

  subject { patch resource_path(resource), params: { resource: new_params } }

  it "updates the title" do
    expect { subject }.to change { resource.reload.title }.from("Old Title").to("New Title")
  end

  it "returns success" do
    subject
    expect(response).to have_http_status(:ok)
  end
end
```

## Factory Usage Guidelines

- **`create`**: When the record must be persisted (testing relationships, associations, queries)
- **`build`**: When you need an unsaved instance (testing validations, before callbacks)
- **`build_stubbed`**: When you need an instance that appears persisted but isn't (fast unit tests)

```ruby
# Use create when testing persistence or queries
let!(:user) { create(:user) }

# Use build when testing validations
let(:invalid_user) { build(:user, email: nil) }

# Use build_stubbed for unit tests that don't touch the database
let(:user) { build_stubbed(:user) }
```

## Before You Write Tests

1. **Understand the feature**: What behavior are you testing?
2. **Identify contexts**: What scenarios/conditions need coverage?
3. **Plan the structure**: What setup is shared? What varies per context?
4. **Write request specs**: Never controller tests
5. **Keep it DRY**: Extract common setup; avoid repetition

## Red Flags to Avoid

- ❌ Creating records inside `it` blocks
- ❌ Making HTTP requests inside `it` blocks
- ❌ Long `it` blocks with multiple assertions on unrelated things
- ❌ Controller tests (use request specs instead)
- ❌ Repeating setup across multiple contexts
- ❌ Using `create` when `build` or `build_stubbed` would suffice

## When to Use This Skill

Use this skill when:
- Writing new RSpec tests for Rails applications
- Converting controller tests to request specs
- Refactoring existing tests to follow this style
- Reviewing test code for style compliance
- Need examples of proper test structure

The agent will analyze your code, understand the feature being tested, and generate well-structured, concise RSpec tests following these exact patterns.

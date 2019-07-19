---
home: true
heroImage: /eventide-icon-132.png
description: 'Microservices and Event Sourcing in Ruby'
actionText: Get Started →
actionLink: /examples/quickstart.md
features:
- title: Microservices
  details: Message-based services hosted in any number of operating system processes or machines, with actor-based pub-sub consumers, component hosting, message dispatching, and handlers
- title: Event Sourcing
  details: Business logic entities projected from event streams with both in-memory, first-level caching and second-level on disk caching
- title: Storage Options
  details: Support for Postgres and EventStore message stores and transports, depending on your performance and scale needs
footer: MIT Licensed | Copyright © 2015-present The Eventide Project
---

- - -

### A Brief Example...

``` ruby
# Account command handler with withdrawal implementation
# Business logic for processing a withdrawal
class Handler
  include Messaging::Handle

  handle Withdraw do |withdraw|
    account_id = withdraw.account_id

    account = store.fetch(account_id)

    unless account.sufficient_funds?(withdraw.amount)
      logger.info('Withdrawal rejected')
      return
    end

    withdrawn = Withdrawn.follow(withdraw)

    time = clock.iso8601
    withdrawn.processed_time = time

    stream_name = stream_name(account_id)

    write.(withdrawn, stream_name)
  end
end

# Withdraw command message
# Send to the account component to effect a withdrawal
class Withdraw
  include Messaging::Message

  attribute :account_id, String
  attribute :amount, Numeric
  attribute :time, String
end

# Withdrawn event message
# Event is written by the handler when a withdrawal is successfully processed
class Withdrawn
  include Messaging::Message

  attribute :account_id, String
  attribute :amount, Numeric
  attribute :time, String
  attribute :processed_time, String
end

# Account entity
# The account component's model object
class Account
  include Schema::DataStructure

  attribute :id, String
  attribute :balance, Numeric, default: 0

  def withdraw(amount)
    self.balance -= amount
  end

  def sufficient_funds?(amount)
    balance >= amount
  end
end

# Account entity projection
# Applies account events to an account entity
class Projection
  include EntityProjection

  entity_name :account

  apply Withdrawn do |withdrawn|
    account.id = withdrawn.account_id

    amount = withdrawn.amount

    account.withdraw(amount)
  end
end

# Account entity store
# Projects an account entity and keeps a cache of the result
class Store
  include EntityStore

  entity Account
  projection Projection
end
```

<div class="hero">
  <p class="action">
    <a href="/examples/" class="nav-link action-button">Go To Examples →</a>
  </p>
</div>

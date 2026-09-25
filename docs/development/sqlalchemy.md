# SQLAlchemy

## Querying Customer Data Warehouses

All customer data warehouse queries are implemented in
[DwhSession](https://github.com/agency-fund/evidential-be/blob/main/src/xngin/apiserver/dwh/dwh_session.py).

## Application Database

Evidential uses synchronous SQLAlchemy with Postgres.

## expire_on_commit

The SQLAlchemy sessions connecting our application to the database are configured with `expire_on_commit` set to False.
This changes the default behavior. We do this because it makes the entity lifecycle more explicit and allows us to avoid
unexpected database queries.

The default behavior is:

```python
# expire_on_commit defaults to True
with Session(engine) as session:
    stmt = select(Experiment).where(Experiment.id == "exp_123")
    result = session.execute(stmt)
    experiment = result.scalar_one_or_none()
    print(f"Experiment name: {experiment.name}")
    experiment.name = "Updated Experiment"
    session.commit()

    # After commit, the experiment object is expired and will automatically be refreshed
    print(
        f"Experiment name after commit: {experiment.name}"
    )  # New DB query happens here!
```

By setting it to false, the behavior changes. Objects after the commit are no-longer invalidated. This means that if
you are reading any database-generated values from newly inserted or updated objects, you must explicitly refresh the
object. Here's how:

```python
with Session(engine, expire_on_commit=False) as session:
    stmt = select(Experiment).where(Experiment.id == "exp_123")
    result = session.execute(stmt)
    experiment = result.scalar_one_or_none()
    print(f"Experiment name: {experiment.name}")
    experiment.name = "Updated Experiment"
    session.commit()

    # After commit, the experiment object is NOT expired
    # No new query is made - we use the in-memory state
    print(f"Experiment name after commit: {experiment.name}")  # No DB query!

    # However, if the database generates values on commit (like updated_at timestamps),
    # you won't see those changes unless you explicitly refresh:
    session.refresh(experiment)
    print(f"Updated timestamp: {experiment.updated_at}")  # Now has the latest DB values
```

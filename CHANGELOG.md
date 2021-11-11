# Changelog

## v1.5.1

### Fix and enhancements

- Generate .env file
- Refresh example templates

## v1.5.0

### Minor compatibility breaks

- Fix variable name `django_log_rotations_frequency` -> `djsite_log_rotations_frequency`

## v1.4.0

### Minor compatibility breaks

- Drop old Celery example configurations

### Fix and enhancements

- Make Celery example configuration compatible with 5+

## v1.3.0

### Minor compatibility breaks

- Do not execute unit-test during setup & update

### Fix and enhancements

- Add unit-test action

## v1.2.0

### Minor compatibility breaks

- Replace update-migrations action by make-migrations
- The make-* actions deploy the release (do not activate it) before running its "action"
- Remove make-migrations from the setup workflow, it has to be executed as an independent action
- Remove `djsite_update_database_migrations` variable accordingly to these behavioral changes

## v1.1.1

### Fix and enhancements

- Fix initial play

## v1.1.0

### Minor compatibilty breaks

- Replace `djsite_version` by `djsite_repository_version`

### Fix and enhancements

- Add name to tasks
- Detect Python executable and make it overridable in config

## v1.0.1

### Fix and enhancements

- Add management of application process (e.g. with uvicorn)
- Make touch reload configurable (`djsite_touch_reload_*` variables)

## v1.0.0

- Initial release

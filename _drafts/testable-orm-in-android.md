--
layout post
title Testable ORM in Android with the Repository Pattern
--
* Test data pollutes real data used for manual testing
** No good way to switch between test and production databases
** Reverting a single transaction would be impractical
* Introduce Repository pattern
** Well defined (external site source)
** Same operations as simple ORM library use (CRUD)
** Allows mocking data model for testing
*** Doesn’t mix with real or manual test data
*** Tests logic w/o database interactions

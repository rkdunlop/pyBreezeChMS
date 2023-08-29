# BREEZE_CHMS_API INTERFACE DETAILS
The module [README](./README.md) has a high-level description of the
`breeze_chms_api` module. Details of the various calls are found here.

## Instantiation
While one _could_ instantiate the `BreezeAPI` class directly, the
preferred mechanism is to use the `breeze_api()` call. 

```python
def breeze_api(breeze_url: str = None,
               api_key: str = None,
               dry_run: bool = False,
               connection: requests.Session = requests.Session(),
               config_name: str = 'breeze_maker.yml',
               **kwargs,
               ) -> BreezeApi:
    """
    Create an instance of a Breeze api object. Parameters needed to configure the api
    can be passed explicitly or loaded from a file using load_config().
    :param breeze_url: Explicitly given url for the Breeze API. (load_config() key 'breeze_url')
    :param api_key: Explicitly given API key for the Breeze API. (load_config() key 'api_key')
    :param dry_run: Just for testing, causes breeze_api to skip all net interactions.
    :param connection: Session if other than default (mostly for testing)
    :param config_name: Alternate load_config() file name.
    :param kwargs: Other parameters used by load_config()
    :return: A BreezeAPI instance
    """
```
Two things are required by `breeze_api()`: The url to connect to your Breeze
instance, and the API key that verifies to Breeze that you are authorized
to connect. While you _can_ pass those explicitly to `breeze_api()`, that
means every application you might use needs to have those values, and
you need to do that in a way that doesn't expose them to potential bad
actors.

The preferred method is to let `breeze_api()` use `load_config()` from the 
[combine_settings](https://pypi.org/project/combine-settings/) module to
load those values from a `breeze_maker.yml` configuration file. (You can
override that file name with the `config_name` parameter.) See the
`combine_settings` module description for where the file should be
placed, but you want it in a place and with permissions so that only
those authorized to use the Breeze API can read it.

## API Calls
`BreezeAPI` is a Python wrapper for the [Breeze API](https://app.breezechms.com/api)
https API. Details of the calls are given there; no attempt is given
here to describe the meaning of the API parameters or the values
returned by each call. Refer to the `Beeze API` description for that.
The following detail the Python interfaces to make those calls.

#### Account Information
```Python
def get_account_summary(self) -> dict:
    """Retrieve the details for a specific account using the API key 
      and URL. It can also work to see if the key and URL are valid.
     :return: JSON object
    """
```
#### People
##### Get Profile Fields
```Python
def get_profile_fields(self) -> List[dict]:
    """
    List profile fields from your database.
    :return: List of descriptors of profile fields
    """
```
##### List People
```Python
def list_people(self, **kwargs) -> List[dict]:
    """
    List people from your database
    :param kwargs: Keyed parameters, all optional:
        limit: If set, number of people to return, If none, will return all people
        offset: Number of people to skip before beginning to return results.
        details: Boolean, if True, return all information, Otherwise just names.
        filter_json: Filter results based on criteria (tags, status, etc.)
                Refer to the list_profile_field response to show values you're
                searching for. Or see the API document for a slightly better
                explanation.
    :return: List of dicts for profiles.
    """
```
This returns all (or a selected subset) of people in your database, with
basic or extended details for each.
##### Get details about a person
```Python
def get_person_details(self, person_id: Union[str, int]) -> dict:
    """
    Retrieve the details for a specific person by their ID.
    :param person_id: Unique id for a person in Breeze database.
    :return: JSON response.
    """
```
##### Add a person
```Python
def add_person(self, **kwargs) -> str:
    """
    Add a new person to the database
    :param kwargs: Keyed parameters:
        first: The first name of the person.
        last: The last name of the person.
        fields_json: JSON string representing an array of fields to update.
                   Each array element must contain field id, field type, response,
                   and in some cases, more information.
                   Obtain such field information from get_profile_fields() or
                   use get_person_details() to see fields that already exist for a specific person.
    :return: JSON response equivalent to get_person_details(). Includes
             the ID of the added person.
    """
```
Note that here, and everywhere following where a parameter
ending in `_json` appears,
the parameter can be a JSON-encoded string, a single Python `dict`, or
a list of `dicts`. The latter two will be automatically converted
for transmission.
##### Update person
```Python
def update_person(self, **kwargs) -> dict:
    """
    Updates the details for a specific person in the database.
    :param kwargs:
      person_id: Unique id for a person in Breeze database.
      fields_json: JSON list of fields to update, similar to add_person().
    :returns: JSON response equivalent to get_person_details(person_id).
    """
```
#### Calendars and Events
##### List calenders
```Python
def list_calendars(self) -> List[dict]:
    """
    Return a list of calendars
    :return: List of descriptions of available calenders
        """
```
##### List Events
```Python
def list_events(self, **kwargs) -> List[dict]:
    """
    Retrieve all events for a given date range.

    :param kwargs: Keyed parameters
      start: Start date; defaults to first day of the current month.
      end: End date; defaults to last day of the current month
    :return: JSON response
"""
```
Dates are in the form YYYY-MM-DD, all numeric.
##### List Event
```Python
def list_event(self, instance_id: Union[str, int]) -> dict:
    """
    Return information about a specific event
    :param instance_id: ID of the event
    :return: json object with event data
    """
```
##### Add Event
```Python
def add_event(self, **kwargs) -> str:
    """
    Add event for given date rage

    :param kwargs: Keyed parameters
      name: Name of event
      starts_on: Start datetimestamp (epoch time)
      ends_on: End datetimestamp (epoch time)
      all_day: boolean
      description: description of event (default none)
      category id: which calendar your event is on (defaults to primary)
      event id: series id
    :return: JSON response
    """
```
##### Check a person in to an event
```Python
def event_check_in(self, person_id, instance_id):
    """
    Checks a person in to an event.
    :param person_id: ID for person in Breeze database
    :param instance_id: ID for event to check in to
    :return: Request response
    """
```
##### Check a person out from an event
```Python
def event_check_out(self, person_id, instance_id):
    """
    Remove the attendance for a person checked into an event.
    :param person_id: Breeze ID for a person in Breeze database.
    :param instance_id: Breeze ID for a person in Breeze database.
    :return: True if check-out succeeds; False if check-out fails.
    """
```
Note: `event_check_out()` differs from `delete_attendance()` in that
it adds a check-out record. (It also adds a check-in if the person
isn't already checked in.) `delete_attendance()` removes all attendance
records for the person.
##### Delete Attendance
```Python
def delete_attendance(self, person_id, instance_id):
    """
    Delete all attendance records for a person from an event
    :param person_id: Id of person to remove
    :param instance_id: Event instance ID
    :return: True if delete succeeds
    """
```
##### List Attendance
```Python
def list_attendance(self, instance_id: Union[str, int], details: bool = False):
    """
    List attendance for an event
    :param instance_id: ID of the event
    :param details: If true, give details
    :return: List of people who attended this event
    """
```
##### List Eligible People
```Python
def list_eligible_people(self, instance_id: Union[int, str]):
    """
    List people eligible for an event
    :param instance_id: ID of the event
    :return: List of eligible people
    """
```
#### Contributions
None of the contributions apis are documented in the generic Breeze API
document, but they're implemented in the original Python implementation.
This implementation was derived from the original Python version, but
documentation is sparse.
##### Add a contribution
```Python
def add_contribution(self, **kwargs) -> str:
    """
    Add a contribution to Breeze.
    :param kwargs: Keyed arguments as follows:
      date: Date of transaction in DD-MM-YYYY format (ie. 24-5-2015)
                 Note: This format is backwards from how Breeze generally does dates.
      name: Name of the person that made the transaction. Used to help match up
            contribution to correct profile within Breeze.
      person_id: The Breeze ID of the donor. If unknown, use UID instead of
                 person id  (ie. 1234567)
      uid: The unique id of the person sent from the giving platform. This
           should be used when the Breeze ID is unknown. Within Breeze a
           user will be able to associate this ID with a given Breeze ID.
           (ie. 9876543)
      email: Email address of the donor. If no person_id is provided, used
             to help automatically match the person to the correct profile.
      street_address: Donor's street address. If person_id is not provided,
                      street_address will be used to help automatically
                      match the person to the correct profile.
      processor: The name of the processor used to send the payment. Used
                 in conjunction with uid. Not needed if using Breeze ID.
                 (ie. SimpleGive, BluePay, Stripe)
      method: The payment method. (ie. Check, Cash, Credit/Debit Online,
              Credit/Debit Offline, Donated Goods (FMV), Stocks (FMV),
              Direct Deposit)
      funds_json: JSON object containing fund names and amounts. This
                  allows splitting fund giving. The ID is optional. If
                  present, it must match an existing fund ID and it will
                  override the fund name. In other words,
                  eg.. [ {
                          'id':'12345',
                          'name':'General Fund',
                          'amount':'100.00'
                        },
                        {
                          'name':'Missions Fund',
                          'amount':'150.00'
                        }
                      ]
      amount: Total amount given. Must match sum of amount in funds_json.
      group: This will create a new batch and enter all contributions with
             the same group into the new batch. Previous groups will be
             remembered and so they should be unique for every new batch.
             Use this if wanting to import into the next batch number in a
             series.
      batch_number: The batch number to import contributions into. Use
                    group instead if you want to import into the next batch
                    number.
      batch_name: The name of the batch. Can be used with batch number or
                  group.
      note: An optional note for the transaction
    :return: Payment id
    :raises: BreezeError on failure to add contribution
    """
```
##### Edit a contribution
```Python
def edit_contribution(self, **kwargs) -> str:
    """
    Edit an existing contribution
    :param kwargs: Keyed parameters
      payment_id: The ID of the payment that should be modified.
      date: Date of transaction in DD-MM-YYYY format (ie. 24-5-2015)
      name: Name of person that made the transaction. Used to help match up
            contribution to correct profile within Breeze.  (ie. John Doe)
      person_id: The Breeze ID of the donor. If unknown, use UID instead of
                 person id  (ie. 1234567)
      uid: The unique id of the person sent from the giving platform. This
           should be used when the Breeze ID is unknown. Within Breeze a
           user will be able to associate this ID with a given Breeze ID.
           (ie. 9876543)
      email: Email address of donor. If no person_id is provided, used to
             help automatically match the person to the correct profile.
             (ie. sample@breezechms.com)
      street_address: Donor's street address. If person_id is not provided,
                      street_address will be used to help automatically
                      match the person to the correct profile.
                      (ie. 123 Sample St)
      processor: The name of the processor used to send the payment. Used
                 in conjunction with uid. Not needed if using Breeze ID.
                 (ie. SimpleGive, BluePay, Stripe)
      method: The payment method. (ie. Check, Cash, Credit/Debit Online,
              Credit/Debit Offline, Donated Goods (FMV), Stocks (FMV),
              Direct Deposit)
      funds_json: JSON string containing fund names and amounts. This
                  allows splitting fund giving. The ID is optional. If
                  present, it must match an existing fund ID and it will
                  override the fund name.
                  ie. [ {
                          'id':'12345',
                          'name':'General Fund',
                          'amount':'100.00'
                        },
                        {
                          'name':'Missions Fund',
                          'amount':'150.00'
                        }
                      ]
      amount: Total amount given. Must match sum of amount in funds_json.
      group: This will create a new batch and enter all contributions with
             the same group into the new batch. Previous groups will be
             remembered and so they should be unique for every new batch.
             Use this if wanting to import into the next batch number in a
             series.
      batch_number: The batch number to import contributions into. Use
                    group instead if you want to import into the next batch
                    number.
      batch_name: The name of the batch. Can be used with batch number or
                  group.
    :returns: Payment iD
    :raises: BreezeError on failure to edit contribution.
    """
```
Note: Since the behavior of this isn't explicitly documented, the author
can only guarantee that the parameters listed here are delivered to Breeze.
There are no guarantees how Breeze handles various cases.
##### Delete Contribution
```Python
def delete_contribution(self, payment_id):
    """
    Delete an existing contribution.
    :param payment_id: The ID of the payment to be deleted
    :return: Payment id
    :raises: BreezeError on failure to delete a contribution.
    """
```
##### List Contributions
```Python
 def list_contributions(self, **kwargs) -> List[Mapping]:
    """
    Retrieve a list of contributions.
    :param kwargs: Set of keyed arguments
      start: Find contributions given on or after a specific date
                  (ie. 2015-1-1); required.
      end: Find contributions given on or before a specific date
                (ie. 2018-1-31); required.
      person_id: ID of person's contributions to fetch. (ie. 9023482)
      include_family: Include family members of person_id (must provide
                      person_id); default: False.
      amount_min: Contribution amounts equal or greater than.
      amount_max: Contribution amounts equal or less than.
      method_ids: List of method IDs.
      fund_ids: List of fund IDs.
      envelope_number: Envelope number.
      batches: List of Batch numbers.
      forms: List of form IDs.
    :return: List of matching contributions as dicts
    :raises: BreezeError on malformed request
    :raises: BreezeBadParameter on missing start or end
    """
```
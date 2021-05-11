# node-challenge

This is the challenge we may discuss in the interview.

We have an API POST endpoin that accepts a JSON in the body with the following structure:

```
{
  "message": "Some message",
  "report_id": 1234,
  "created_by": "123456",
  "is_pilot_created": true,
  "is_pilot_visible": false,
  "claim_report_note_id": "1234_567_777"
}
```
Here is the function that processes the POST request:

```
const createPilotReportNote = (pool) => {
  return async (request, response, next) => {
    const cols = ['message', 'report_id', 'created_by', 'is_pilot_created', 'is_pilot_visible', 'claim_report_note_id'];
    let values = cols.map((k) => request.body[k]);

    const valuesIdxs = cols.map((v, i) => `$${i + 1}`).join(', ');

    const query = `INSERT INTO report_notes(${cols.join(', ')}) values(${valuesIdxs}) RETURNING *`;
    
    try {
      const results = await pool.query(query, values);

      response.status(200).json(results.rows[0]);
    } catch (e) {
      next(e);
    }
  };
};
```

In this function `request.body` is object representing the body of the request.  For example:

```
// EXAMPLE 1
request.body = {
  message: "Some message",
  report_id: 1234,
  created_by: "123456",
  is_pilot_created: true,
  is_pilot_visible: false,
  claim_report_note_id: "1234_567_777"
}
```

The above input results in `query` equaling:

`INSERT INTO report_notes(message, report_id, created_by, is_pilot_created, is_pilot_visible, claim_report_id) values($!, $2, $3, $4, $5, $6) RETURNING *`

As written, the above `INSERT` statement is generated regardless of the elements provided in the body. Any missing fields will have a `NULL` value inserted into the database (our database driver converts `undefined` into `null`).

For example:
```
// EXAMPLE 2
request.body = {
  message: "Some message",
  report_id: 1234,
  created_by: "123456",
  claim_report_note_id: "1234_567_777"
}
```
Results in the same `query`.  This will cause an attempt to insert `NULL` into the `is_pilot_created` and `is_pilot_visible` fields. Since these fields do not accept `NULL`an error will be thrown.

Your challenge is to rewrite the above code to make all of the fields optional.  If the field is not provided in the body of the request, it should not be included in the `INSERT` statement.  So the input from EXAMPLE 2 should generate the following `query`:

`INSERT INTO report_notes(message, report_id, created_by, claim_report_id) values($!, $2, $3, $4) RETURNING *`

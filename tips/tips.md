## Process when creating or saving an issue

```javascript
document.getElementById('issue-form').addEventListener('submit', (e) => {
  if (!confirm('Are you sure?')) {
    // Cancel the form submission
    e.stopPropagation();
    e.preventDefault();
    return false;
  }
}, true); // Use capturing phase to prevent double-submit protection from being triggered when user cancels
```

## Judge the creation and editing of an issue

```javascript
if ($('form.new_issue').length === 1) {
  // new issue
  console.log('new');
}

if ($('form.edit_issue').length === 1) {
  // edit issue
  console.log('edit');
}
```

## Check the roles

```javascript
const hasRoleId3 = ViewCustomize.context.project.roles.some(function(role) { return role.id == 3 });
```

* https://github.com/onozaty/redmine-view-customize#viewcustomizecontext-javascript

## Check the value before the change

The previous value should be saved.

```javascript
const target = '#issue_assigned_to_id';
let prevValue = $(target).val();

$(target).on('change', function() {
  console.log('old:' + prevValue + ' new:' + $(target).val());
  prevValue = $(target).val();
});
```

For status and other forms that replace the form itself when changed, the following must be done.  
"Insertion position" should be `Head of all pages`.

```javascript
const target = '#issue_status_id';
let prevValue = $(target).val();

$('#all_attributes').on('change', target, function() {
  console.log('old:' + prevValue + ' new:' + $(target).val());
  prevValue = $(target).val();
});
```

## Using the REST API to create an issue

By using the Redmine REST API, you can create an issue.

* [Rest Issues \- Redmine](https://www.redmine.org/projects/redmine/wiki/Rest_Issues)

```javascript
const issue = {
  'issue': {
    'project_id': 1,
    'tracker_id': 1,
    'subject': 'XYZ: ' + new Date().toISOString()
  }
};

$.ajax({
  type: 'POST',
  url: '/issues.json',
  headers: {
    'X-Redmine-API-Key': ViewCustomize.context.user.apiKey
  },
  dataType: 'json',
  contentType: 'application/json',
  data: JSON.stringify(issue)
})
.done(function(response) {
  alert('Succeed. issue Id: ' + response.issue.id);
})
.fail(function() {
  alert('Failed.');
});
```

When creating parents and children together, do the following.

```javascript
const parentIssue = {
  'issue': {
    'project_id': 1,
    'tracker_id': 1,
    'subject': 'Parent: ' + new Date().toISOString()
  }
};

const childIssue = {
  'issue': {
    'project_id': 1,
    'tracker_id': 1,
    'subject': 'Child: ' + new Date().toISOString()
  }
};

$.ajax({
  type: 'POST',
  url: '/issues.json',
  headers: {
    'X-Redmine-API-Key': ViewCustomize.context.user.apiKey
  },
  dataType: 'json',
  contentType: 'application/json',
  data: JSON.stringify(parentIssue)
})
.then(function(response) {
  childIssue.issue.parent_issue_id = response.issue.id;
  return $.ajax({
    type: 'POST',
    url: '/issues.json',
    headers: {
      'X-Redmine-API-Key': ViewCustomize.context.user.apiKey
    },
    dataType: 'json',
    contentType: 'application/json',
    data: JSON.stringify(childIssue)
  });
})
.done(function(response) {
  alert('Succeed. parent issue Id: ' + response.issue.parent.id + ', child issue Id: ' + response.issue.id);
})
.fail(function() {
  alert('Failed.');
});
```

## Using the REST API to fetch issue

By using Redmine REST API, you can get information about issues that are not displayed on the screen.

* [Rest Issues \- Redmine](https://www.redmine.org/projects/redmine/wiki/Rest_Issues)

However, this can be very complicated.  
It is recommended that you use it only if you understand how it works.

```javascript
const issueId = 1;

$.ajax({
  type: 'GET',
  url: '/issues/' + issueId + '.json',
  headers: {
    'X-Redmine-API-Key': ViewCustomize.context.user.apiKey
  },
  contentType: 'application/json'
})
.done(function(data){
  const issue = data.issue;
  console.log('Subject:' + issue.subject);
  console.log('Status:' + issue.status.id);
})
.fail(function(jqXHR, textStatus, errorThrown){
  console.log(jqXHR.status);
});
```

This is an example of data that can be fetched by the API.

```json
{
  "issue": {
    "id": 1,
    "project": {
      "id": 1,
      "name": "a"
    },
    "tracker": {
      "id": 2,
      "name": "Feature"
    },
    "status": {
      "id": 6,
      "name": "Rejected"
    },
    "priority": {
      "id": 4,
      "name": "Urgent"
    },
    "author": {
      "id": 1,
      "name": "Redmine Admin"
    },
    "assigned_to": {
      "id": 8,
      "name": "Ai Hoshino"
    },
    "fixed_version": {
      "id": 1,
      "name": "v1.0.0"
    },
    "parent": {
      "id": 2
    },
    "subject": "Added option to always generate API key",
    "description": "",
    "start_date": "2021-09-20",
    "due_date": null,
    "done_ratio": 90,
    "is_private": false,
    "estimated_hours": 3,
    "total_estimated_hours": 3,
    "spent_hours": 0,
    "total_spent_hours": 0,
    "custom_fields": [
      {
        "id": 5,
        "name": "Values",
        "multiple": true,
        "value": [
          "10",
          "11"
        ]
      },
      {
        "id": 11,
        "name": "Number",
        "value": "1"
      },
      {
        "id": 13,
        "name": "Date",
        "value": "2022-01-23"
      }
    ],
    "created_on": "2021-09-20T08:30:17Z",
    "updated_on": "2022-01-23T13:56:56Z",
    "closed_on": "2021-10-03T04:40:03Z"
  }
}
```

## Get the value of a issue field

```javascript
const val = $('#issue_custom_field_values_1').val();
```

In the case of a checkbox, it is as follows.

```javascript
// checked: 1, unchecked: undefined
const val = $('#issue_custom_field_values_1:checked').val();
```

```javascript
// checked: true, unchecked: false
const val = $('#issue_custom_field_values_1').prop('checked');
```

In the case of a radio, it is as follows.

```javascript
const val = $('input[name="issue[custom_field_values][1]"]:checked').val();
```

The label of a custom field in key/value list format is as follows.

```javascript
const label = $('#issue_custom_field_values_1 > option:selected').text();
```

## Get the value of a project custom field

Use `ViewCustomize.context`.

* https://github.com/onozaty/redmine-view-customize#viewcustomizecontext-javascript

```javascript
const field = ViewCustomize.context.project.customFields.find(x => x.id == 1);
const value = (field == undefined) ? undefined : field.value;
```

## Set value for selectbox using the select2 library

Some Redmine plugins make use of the select2 library.  

The following considerations must be made when setting value in selectbox using the select2 library.

* https://select2.org/programmatic-control/add-select-clear-items#selecting-options

For example, if you want to set a value for an assignee, issue a change event afterwards.

```javascript
$('#issue_assigned_to_id').val('1');
$('#issue_assigned_to_id').trigger('change');
```

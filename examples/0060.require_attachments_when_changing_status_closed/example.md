# Require attachments when changing status to closed

Require attachments when changing status to closed.  
ステータスを終了に変える時に、添付ファイルを必須にします。

## Setting

### Path Pattern

None

### Insert Position

Bottom of issue detail
<!-- 
Head of all pages
Bottom of issue form
Bottom of issue detail
Bottom of all pages
-->

### Code

JavaScript
<!--
JavaScript
CSS
HTML
-->

```javascript
$(function() {
  const beforeStatusId = $('#issue_status_id').val();

  document.getElementById('issue-form').addEventListener('submit', function(e) {
    const afterStatusId = $('#issue_status_id').val();

    if (afterStatusId != beforeStatusId && afterStatusId == '5') { // 5: Closed
      if ($('#new-attachments .attachments_fields input.filename').length == 0) { // Check zero attachments

        alert('If status is changed to Closed, attachments are required.');

        // Cancel the form submission
        e.stopPropagation();
        e.preventDefault();
        return false;
      }
    }
  }, true); // Use capturing phase to prevent double-submit protection from being triggered when user cancels
});
```

## Result

![result](./result.gif)

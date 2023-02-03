# Overview
This is full-stack web application using .NET MVC connected to a SQL database through Visual Studio. “MinesweeperWeb” is exactly as it sounds, the classic Minesweeper game hosted on a web server.

![Minesweeper game page](https://user-images.githubusercontent.com/78822631/216502178-fd79b84d-3c29-4d92-99ba-6aeb374e09e4.jpg)

# Documentation

### Scrum Planning

![Scrum Planning](https://user-images.githubusercontent.com/78822631/216502763-00229dae-2837-4cdf-bf6d-8d44e9aaa870.png)

### Burndown Chart

![Burndown Chart](https://user-images.githubusercontent.com/78822631/216502799-cf9828b9-f46c-4714-be0d-0a5e9c43af2b.png)

### Sitemap

![Sitemap](https://user-images.githubusercontent.com/78822631/216502862-3a15ec1c-1962-4bb8-bc1e-8842f3037376.png)

# Code Snippets

### MVC Architecture

### AJAX

```
// Updates a minesweeper cell's display on click
function doButtonUpdate(buttonRow, buttonCol, urlString) {
    // Send POST request to urlString parameter and send buttonRow + buttonCol as request parameters
    $.ajax({
        datatype: "json",
        method: 'POST',
        url: urlString,
        data: {
            "buttonRow": buttonRow,
            "buttonCol": buttonCol
        },
        // If request returns 200, update display
        success: function (data) {
            console.log(data);
            $("[value='" + buttonRow + "," + buttonCol + "']").replaceWith($(data));
        }
    });
};
```

### .NET Page Security

```

```

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
The 'BoardModel' Model represents the game board used for Minesweeper. The View uses the 'BoardModel' to give the user options for how they would like to initialize the board. The 'HomeController' Controller directs the user to this View, so that they may interact with the Model. Demonstrates proficiency in MVC architecture.

*Model*
```
// Class that represents game board containing cells to click
public class BoardModel
{
    // Width of board
    [Required]
    [DisplayName("Board Width")]
    [Range(5, 30)]
    public int Width { get; set; }
    // Height of board
    [Required]
    [DisplayName("Board Height")]
    [Range(5, 30)]
    public int Height { get; set; }
    // Percentage of cells that will become bombs
    [Required]
    [DisplayName("Game Difficulty")]
    public double Difficulty { get; set; }
    // Array used to store multiple instances of the Cell class
    public CellModel[,] CellMatrix { get; set; }


    // Parameterized constructor
    public BoardModel(int width, int height, double difficulty)
    {
        this.Width = width;
        this.Height = width;

        this.Difficulty = difficulty;
        this.CellMatrix = new CellModel[width, width];

        // Fills board matrix with Cell objects
        for (int row = 0; row < width; row++)
        {
            for (int column = 0; column < width; column++)
            {
                CellMatrix[row, column] = new CellModel(row, column);
            }
        }
    }

    // Non-parameterized constructor - values assigned by entry form
    public BoardModel()
    {
        this.CellMatrix = new CellModel[Width, Height];

        // Fills board matrix with Cell objects
        for (int row = 0; row < Height; row++)
        {
            for (int column = 0; column < Width; column++)
            {
                CellMatrix[row, column] = new CellModel(row, column);
            }
        }
    }
}
```

*View*
```
@model MinesweeperWeb.Models.BoardModel

<h4>Set Board Parameters</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form asp-action="InitializeGame">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Width" class="control-label">Size</label>
                <input asp-for="Width" class="form-control" />
                <span asp-validation-for="Width" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Difficulty" class="control-label"></label>
                <select class="form-control" aria-label="Default select example" asp-for="Difficulty">
                    <option value="0.1">Easy</option>
                    <option value="0.2" selected>Medium</option>
                    <option value="0.3">Hard</option>
                </select>

                <span asp-validation-for="Difficulty" class="text-danger"></span>

            </div>
            <div class="form-group">
                <input type="submit" value="Begin Game" class="btn btn-outline-primary" />
            </div>  
        </form>

        <form asp-action="DisplaySavedGames">
            <div class="form-group">
                <input type="submit" value="Saved Games" class="btn btn-primary" />
            </div>
        </form>
    </div>
</div>
```

*Controller*
```
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;

    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }

    public IActionResult Index()
    {
        if(GameController.GameState == "INITIAL")
        {
            return RedirectToAction("Index", "Login");
        }
        else
        {
            return View();
        }
    }

    public IActionResult Privacy()
    {
        return View();
    }

    [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
    public IActionResult Error()
    {
        return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
    }
}

```

### AJAX
AJAX functionality that updates the game board upon clicking a cell. Demonstrates ability to use AJAX for partially reloading web pages via API requests.
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

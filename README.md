# Minesweeper
Classic Minesweeper Game in Unity
## Overview

![Alt Text](https://github.com/matthewsgordon/minesweeper/blob/main/Unity_4e94rHfdWx.gif)

## Design
### Board Class
This class handles the 2D board as a Unity grid.
```C#
using UnityEngine;
using UnityEngine.Tilemaps;

public class Board : MonoBehaviour
{
    public Tilemap tilemap {get; private set;}

    public Tile tileUnknown;
    public Tile tileEmpty;
    public Tile tileMine;
    public Tile tileExploded;
    public Tile tileFlag;
    public Tile tileNum1;
    public Tile tileNum2;
    public Tile tileNum3;
    public Tile tileNum4;
    public Tile tileNum5;
    public Tile tileNum6;
    public Tile tileNum7;
    public Tile tileNum8;


    private void Awake()
    {
        tilemap = GetComponent<Tilemap>();
    }

    public void Draw(Cell[,] state)
    {
        int width = state.GetLength(0);
        int height = state.GetLength(1);
        
        for (int x = 0; x < width; x++)
        {
            for (int y = 0; y < height; y++)
            {
                Cell cell = state [x, y];
                tilemap.SetTile(cell.position, GetTile(cell));
            }
        }
    }
    private Tile GetTile(Cell cell)
    {
        if (cell.isRevealed) {
            return GetRevealedTile(cell);
        }
        else if(cell.isFlagged){
            return tileFlag;
        }
        else {
            return tileUnknown;
        }
    }

    private Tile GetRevealedTile (Cell cell)
    {
        switch (cell.type)
        {
            case Cell.Type.Empty: return tileEmpty;
            case Cell.Type.Mine: return cell.isMine ? tileExploded : tileMine;
            case Cell.Type.Number: return GetNumberTile(cell); 
            default: return null;
        }
    }
    private Tile GetNumberTile (Cell cell)
    {
        switch(cell.number)
        {
            case 1: return tileNum1;
            case 2: return tileNum2;
            case 3: return tileNum3;
            case 4: return tileNum4;
            case 5: return tileNum5;
            case 6: return tileNum6;
            case 7: return tileNum7;
            case 8: return tileNum8;
            default: return null; 
           
        }
    }

}
```
### Cell Class
This class is data structure for a cell on the board.
```C#
using UnityEngine;

public struct Cell 
{
    public enum Type 
    {
        Invalid,
        Empty,
        Mine,
        Number,
    }

    public Vector3Int position;
    public Type type;
    public int number;
    public bool isRevealed;
    public bool isFlagged;
    public bool isMine;
}
```

### Game Class
This class handles the game logic.
```C#
using UnityEngine;

public class Game : MonoBehaviour
{
    public int width = 16;
    public int height = 16;
    public int mines = 32;
    private Board board;
    public bool gameOver;

    private Cell[,] state;

    private void Awake()
    {
        board = GetComponentInChildren<Board>();
    }

    private void Start()
    {
        NewGame();
    }

    private void NewGame()
    {
        state = new Cell[width, height];

        GenerateCells();
        GenerateMines();
        GenerateNumbers();

        Camera.main.transform.position = new Vector3(width / 2f, height / 2f, -10f);

        board.Draw(state);
    }

    private void GenerateCells()
    {
        for (int x = 0; x < width; x++)
        {
            for (int y = 0; y < height; y++)
            {
                Cell cell = new Cell();
                cell.position = new Vector3Int (x, y ,0);
                cell.type = Cell.Type.Empty;
                state[x, y] = cell;
            }
            
        }
    }
    private void GenerateMines()
    {
        for (int i = 0; i < mines; i++)
        {
            int x = Random.Range(0, width);
            int y = Random.Range(0, height);

            while (state [x, y].type == Cell.Type.Mine)
            {
                x++;

                if(x >= width)
                {
                    x = 0;
                    y++;

                    if (y >= height )
                    {
                        y = 0;
                    }
                }
                

            }
            state[x, y].type = Cell.Type.Mine;

       
        }
    }
    private void GenerateNumbers()
    {
         for (int x = 0; x < width; x++)
        {
            for (int y = 0; y < height; y++)
            {
                Cell cell = state[x, y];
                
                if( cell.type == Cell.Type.Mine)
                {
                    continue;
                }
                cell.number = CountMines(x, y);

                if (cell.number > 0)
                {
                    cell.type = Cell.Type.Number;
                }
                state[x, y] = cell;


            }
    }
 }
    private int CountMines(int cellX, int cellY)
    {
        int count = 0; 

        for(int adjacentX = -1; adjacentX <= 1; adjacentX++)
        {
            for(int adjacentY = -1; adjacentY <= 1; adjacentY++)
            {
                if (adjacentX == 0 && adjacentY == 0)
                {
                    continue;
                }
                
                int x = cellX + adjacentX;
                int y = cellY + adjacentY;

               

                if (GetCell(x,y).type == Cell.Type.Mine)
                {
                    count++;
                }
            }
        }
        return count;
    }

    private void Update()
    {
        if(Input.GetMouseButtonDown(1))
        {
            Flag();
            
        }
        else if(Input.GetMouseButtonDown(0))
        {
            Reveal();
        } 
    }
    private void Flag()
    {
        Vector3 worldPosition = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        Vector3Int cellPosition = board.tilemap.WorldToCell(worldPosition);
        Cell cell = GetCell(cellPosition.x, cellPosition.y);

        if (cell.type == Cell.Type.Invalid || cell.isRevealed)
        {
            return;
        }
        cell.isFlagged = !cell.isFlagged;
        state[cellPosition.x, cellPosition.y] = cell;
        board.Draw(state);

    }

     private void Reveal()
    {
        Vector3 worldPosition = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        Vector3Int cellPosition = board.tilemap.WorldToCell(worldPosition);
        Cell cell = GetCell(cellPosition.x, cellPosition.y);

        if (cell.type == Cell.Type.Invalid || cell.isRevealed || cell.isFlagged)
        {
            return;
        }

        switch (cell.type)
        {
            case Cell.Type.Mine:
            Explode(cell);
            break;

            case Cell.Type.Empty:
            Flood(cell);
            Win();
            break;

            default:
            cell.isRevealed = true;
            state[cellPosition.x, cellPosition.y] = cell;
            Win();
            break;
        }        
        board.Draw(state);          

    }

    private void Flood(Cell cell)
        {
            if(cell.isRevealed) return;
            if(cell.type == Cell.Type.Mine || cell.type == Cell.Type.Invalid) return;

            cell.isRevealed = true;
            state[cell.position.x, cell.position.y] = cell;

            if(cell.type == Cell.Type.Empty) 
            {
                Flood(GetCell(cell.position.x - 1, cell.position.y));
                Flood(GetCell(cell.position.x + 1, cell.position.y));
                Flood(GetCell(cell.position.x, cell.position.y - 1));
                Flood(GetCell(cell.position.x, cell.position.y + 1));

            }
        }
    private void Explode(Cell cell)
    {
        Debug.Log("Game Over");
        gameOver = true;
        Application.Quit();

        cell.isRevealed = true;
        cell.isMine = true;
        state[cell.position.x , cell.position.y] = cell;

        for(int x = 0; x < width; x ++)
        {
            for(int y = 0; y < height; y ++)
            {
                cell = state[x, y];

                if (cell.type == Cell.Type.Mine)
                {
                    cell.isRevealed = true;
                    state[x , y] = cell;
                }
            }
        }
    }

    private Cell GetCell(int x, int y)
    {
        if(IsValid(x, y))
        {
            return state[x, y];
        }
        else 
        {
            return new Cell();
        }
    }
    
    private void Win()
    {
        for(int x = 0; x < width; x++)
        {
            for(int y = 0; y < height; y++)
            {
                Cell cell = state [x,y];
                if (cell.type != Cell.Type.Mine && !cell.isRevealed)
                {
                    return;
                }
            }
        }
        Debug.Log("You win");
        gameOver = true;

        for(int x = 0; x < width; x ++)
        {
            for(int y = 0; y < height; y ++)
            {
                Cell cell = state[x, y];

                if (cell.type == Cell.Type.Mine)
                {
                    cell.isFlagged = true;
                    state[x , y] = cell;
                }
            }
        }
    }

    private bool IsValid(int x, int y)
    {
        return x >= 0 && x < width && y >= 0 && y < height;
    }   
}
```

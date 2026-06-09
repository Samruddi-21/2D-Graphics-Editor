#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define ROWS 25
#define COLS 50
#define MAX_SHAPES 100
#define BG_CHAR '_'
#define DRAW_CHAR '*'

typedef enum {
    SHAPE_LINE,
    SHAPE_RECTANGLE,
    SHAPE_CIRCLE,
    SHAPE_TRIANGLE
} ShapeType;

typedef struct {
    int id;
    ShapeType type;
    union {
        struct { int x1, y1, x2, y2; } line;
        struct { int x1, y1, x2, y2; } rect;
        struct { int xc, yc, r; } circle;
        struct { int x1, y1, x2, y2, x3, y3; } triangle;
    } data;
} Shape;

// Global variables
char canvas[ROWS][COLS];
Shape shapes[MAX_SHAPES];
int shape_count = 0;
int next_id = 1;

// Function declarations
void init_canvas();
void set_pixel(int x, int y);
void draw_line(int x1, int y1, int x2, int y2);
void draw_rectangle(int x1, int y1, int x2, int y2);
void draw_circle_points(int xc, int yc, int x, int y);
void draw_circle(int xc, int yc, int r);
void draw_triangle(int x1, int y1, int x2, int y2, int x3, int y3);
void render_canvas();
void display_canvas();
void list_shapes();
int delete_shape(int id);
int read_int(const char *prompt, int min_val, int max_val);
void add_shape_menu();
void delete_shape_menu();
void show_title();

int main() {
    init_canvas();
    int choice;

    while (1) {
        show_title();
        printf("1. Add a Shape\n");
        printf("2. Delete a Shape\n");
        printf("3. Display the Picture\n");
        printf("4. Clear Canvas (Delete All Shapes)\n");
        printf("5. Exit\n");
        printf("--------------------------------------------------\n");
        
        choice = read_int("Enter your choice (1-5): ", 1, 5);

        switch (choice) {
            case 1:
                add_shape_menu();
                break;
            case 2:
                delete_shape_menu();
                break;
            case 3:
                render_canvas();
                display_canvas();
                break;
            case 4:
                shape_count = 0;
                render_canvas();
                printf("\nCanvas cleared successfully!\n");
                display_canvas();
                break;
            case 5:
                printf("\nExiting 2D Graphics Editor. Goodbye!\n");
                return 0;
            default:
                printf("\nInvalid choice. Please try again.\n");
        }
    }
    return 0;
}

// Initialize the 2D canvas with background characters
void init_canvas() {
    for (int y = 0; y < ROWS; y++) {
        for (int x = 0; x < COLS; x++) {
            canvas[y][x] = BG_CHAR;
        }
    }
}

// Safely plot a point on the canvas (clips out of bounds points)
void set_pixel(int x, int y) {
    if (x >= 0 && x < COLS && y >= 0 && y < ROWS) {
        canvas[y][x] = DRAW_CHAR;
    }
}

// Bresenham's line algorithm
void draw_line(int x1, int y1, int x2, int y2) {
    int dx = abs(x2 - x1);
    int dy = -abs(y2 - y1);
    int sx = (x1 < x2) ? 1 : -1;
    int sy = (y1 < y2) ? 1 : -1;
    int err = dx + dy;
    int e2;

    while (1) {
        set_pixel(x1, y1);
        if (x1 == x2 && y1 == y2) {
            break;
        }
        e2 = 2 * err;
        if (e2 >= dy) {
            err += dy;
            x1 += sx;
        }
        if (e2 <= dx) {
            err += dx;
            y1 += sy;
        }
    }
}

// Rectangle drawing by drawing 4 lines
void draw_rectangle(int x1, int y1, int x2, int y2) {
    draw_line(x1, y1, x2, y1); // Top
    draw_line(x1, y2, x2, y2); // Bottom
    draw_line(x1, y1, x1, y2); // Left
    draw_line(x2, y1, x2, y2); // Right
}

// Helper for octant mirroring in circle algorithm
void draw_circle_points(int xc, int yc, int x, int y) {
    set_pixel(xc + x, yc + y);
    set_pixel(xc - x, yc + y);
    set_pixel(xc + x, yc - y);
    set_pixel(xc - x, yc - y);
    set_pixel(xc + y, yc + x);
    set_pixel(xc - y, yc + x);
    set_pixel(xc + y, yc - x);
    set_pixel(xc - y, yc - x);
}

// Midpoint/Bresenham circle drawing algorithm
void draw_circle(int xc, int yc, int r) {
    if (r < 0) return;
    int x = 0;
    int y = r;
    int d = 3 - 2 * r;
    draw_circle_points(xc, yc, x, y);
    while (y >= x) {
        x++;
        if (d > 0) {
            y--;
            d = d + 4 * (x - y) + 10;
        } else {
            d = d + 4 * x + 6;
        }
        draw_circle_points(xc, yc, x, y);
    }
}

// Triangle drawing by drawing 3 lines
void draw_triangle(int x1, int y1, int x2, int y2, int x3, int y3) {
    draw_line(x1, y1, x2, y2);
    draw_line(x2, y2, x3, y3);
    draw_line(x3, y3, x1, y1);
}

// Rebuilds the canvas from scratch using current active shape list
void render_canvas() {
    init_canvas();
    for (int i = 0; i < shape_count; i++) {
        Shape s = shapes[i];
        switch (s.type) {
            case SHAPE_LINE:
                draw_line(s.data.line.x1, s.data.line.y1, s.data.line.x2, s.data.line.y2);
                break;
            case SHAPE_RECTANGLE:
                draw_rectangle(s.data.rect.x1, s.data.rect.y1, s.data.rect.x2, s.data.rect.y2);
                break;
            case SHAPE_CIRCLE:
                draw_circle(s.data.circle.xc, s.data.circle.yc, s.data.circle.r);
                break;
            case SHAPE_TRIANGLE:
                draw_triangle(s.data.triangle.x1, s.data.triangle.y1, s.data.triangle.x2, s.data.triangle.y2, s.data.triangle.x3, s.data.triangle.y3);
                break;
        }
    }
}

// Print canvas contents enclosed in a nice frame with guide axes
void display_canvas() {
    printf("\n");
    // Top border
    printf("   +");
    for (int x = 0; x < COLS; x++) {
        printf("-");
    }
    printf("+\n");

    // Rows
    for (int y = 0; y < ROWS; y++) {
        printf("%2d |", y);
        for (int x = 0; x < COLS; x++) {
            printf("%c", canvas[y][x]);
        }
        printf("|\n");
    }

    // Bottom border
    printf("   +");
    for (int x = 0; x < COLS; x++) {
        printf("-");
    }
    printf("+\n");

    // Horizontal guide markings
    printf("    ");
    for (int x = 0; x < COLS; x++) {
        if (x % 5 == 0) {
            printf("%-5d", x);
        }
    }
    printf("\n\n");
}

// List all active shapes and their descriptions
void list_shapes() {
    if (shape_count == 0) {
        printf("No active shapes on canvas.\n");
        return;
    }
    printf("Current shapes:\n");
    for (int i = 0; i < shape_count; i++) {
        Shape s = shapes[i];
        switch (s.type) {
            case SHAPE_LINE:
                printf("  [ID %d] Line from (%d, %d) to (%d, %d)\n", s.id, s.data.line.x1, s.data.line.y1, s.data.line.x2, s.data.line.y2);
                break;
            case SHAPE_RECTANGLE:
                printf("  [ID %d] Rectangle top-left (%d, %d), bottom-right (%d, %d)\n", s.id, s.data.rect.x1, s.data.rect.y1, s.data.rect.x2, s.data.rect.y2);
                break;
            case SHAPE_CIRCLE:
                printf("  [ID %d] Circle center (%d, %d), radius %d\n", s.id, s.data.circle.xc, s.data.circle.yc, s.data.circle.r);
                break;
            case SHAPE_TRIANGLE:
                printf("  [ID %d] Triangle points (%d, %d), (%d, %d), (%d, %d)\n", s.id, s.data.triangle.x1, s.data.triangle.y1, s.data.triangle.x2, s.data.triangle.y2, s.data.triangle.x3, s.data.triangle.y3);
                break;
        }
    }
}

// Delete a shape by ID, shift remaining elements to pack array
int delete_shape(int id) {
    int found_index = -1;
    for (int i = 0; i < shape_count; i++) {
        if (shapes[i].id == id) {
            found_index = i;
            break;
        }
    }
    if (found_index == -1) {
        return 0; // Not found
    }
    
    // Shift elements
    for (int i = found_index; i < shape_count - 1; i++) {
        shapes[i] = shapes[i + 1];
    }
    shape_count--;
    return 1;
}

// Title banner printer helper
void show_title() {
    printf("\n==================================================\n");
    printf("            2D GRAPHICS CANVAS EDITOR             \n");
    printf("==================================================\n");
}

// Robust input reader for console integers
int read_int(const char *prompt, int min_val, int max_val) {
    int val;
    char buffer[100];
    while (1) {
        printf("%s", prompt);
        if (fgets(buffer, sizeof(buffer), stdin) == NULL) {
            continue;
        }
        
        // Trim trailing newline
        buffer[strcspn(buffer, "\n")] = '\0';
        
        // Skip empty entries
        if (strlen(buffer) == 0) {
            continue;
        }
        
        char *endptr;
        long parsed = strtol(buffer, &endptr, 10);
        if (endptr == buffer || *endptr != '\0') {
            printf("Invalid format. Please enter an integer.\n");
            continue;
        }
        if (parsed < min_val || parsed > max_val) {
            printf("Value out of range [%d, %d]. Please try again.\n", min_val, max_val);
            continue;
        }
        val = (int)parsed;
        break;
    }
    return val;
}

// Menu handler for adding a shape
void add_shape_menu() {
    if (shape_count >= MAX_SHAPES) {
        printf("\nError: Maximum shape limit reached (%d)!\n", MAX_SHAPES);
        return;
    }

    printf("\n--- Add a Shape ---\n");
    printf("1. Line\n");
    printf("2. Rectangle\n");
    printf("3. Circle\n");
    printf("4. Triangle\n");
    printf("-------------------\n");
    
    int type = read_int("Select shape type (1-4): ", 1, 4);
    Shape s;
    s.id = next_id++;

    switch (type) {
        case 1:
            s.type = SHAPE_LINE;
            printf("Enter Line parameters:\n");
            s.data.line.x1 = read_int("  Start X (0-49): ", 0, COLS - 1);
            s.data.line.y1 = read_int("  Start Y (0-24): ", 0, ROWS - 1);
            s.data.line.x2 = read_int("  End X (0-49): ", 0, COLS - 1);
            s.data.line.y2 = read_int("  End Y (0-24): ", 0, ROWS - 1);
            break;
        case 2:
            s.type = SHAPE_RECTANGLE;
            printf("Enter Rectangle parameters:\n");
            s.data.rect.x1 = read_int("  Top-Left X (0-49): ", 0, COLS - 1);
            s.data.rect.y1 = read_int("  Top-Left Y (0-24): ", 0, ROWS - 1);
            s.data.rect.x2 = read_int("  Bottom-Right X (0-49): ", 0, COLS - 1);
            s.data.rect.y2 = read_int("  Bottom-Right Y (0-24): ", 0, ROWS - 1);
            break;
        case 3:
            s.type = SHAPE_CIRCLE;
            printf("Enter Circle parameters:\n");
            s.data.circle.xc = read_int("  Center X (0-49): ", 0, COLS - 1);
            s.data.circle.yc = read_int("  Center Y (0-24): ", 0, ROWS - 1);
            s.data.circle.r = read_int("  Radius (0-25): ", 0, 25);
            break;
        case 4:
            s.type = SHAPE_TRIANGLE;
            printf("Enter Triangle parameters:\n");
            s.data.triangle.x1 = read_int("  Vertex 1 X (0-49): ", 0, COLS - 1);
            s.data.triangle.y1 = read_int("  Vertex 1 Y (0-24): ", 0, ROWS - 1);
            s.data.triangle.x2 = read_int("  Vertex 2 X (0-49): ", 0, COLS - 1);
            s.data.triangle.y2 = read_int("  Vertex 2 Y (0-24): ", 0, ROWS - 1);
            s.data.triangle.x3 = read_int("  Vertex 3 X (0-49): ", 0, COLS - 1);
            s.data.triangle.y3 = read_int("  Vertex 3 Y (0-24): ", 0, ROWS - 1);
            break;
    }

    shapes[shape_count++] = s;
    printf("\nShape added successfully!\n");
    render_canvas();
    display_canvas();
}

// Menu handler for deleting a shape
void delete_shape_menu() {
    printf("\n--- Delete a Shape ---\n");
    if (shape_count == 0) {
        printf("No active shapes to delete.\n");
        return;
    }
    list_shapes();
    printf("----------------------\n");
    int id = read_int("Enter shape ID to delete: ", 1, next_id - 1);
    
    if (delete_shape(id)) {
        printf("\nShape with ID %d deleted successfully!\n", id);
        render_canvas();
        display_canvas();
    } else {
        printf("\nError: Shape with ID %d not found.\n", id);
    }
}

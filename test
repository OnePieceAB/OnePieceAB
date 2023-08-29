#include <iostream>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <sys/mman.h>
#include <linux/fb.h>
#include <sys/ioctl.h>
#include <stdio.h>
#include <linux/input.h>
#include <stdlib.h>
#define PATh "/bmp/"

using namespace std;
class catalogue
{
public:
    char catalogSet[100][100];
    char bmp_color[800 * 480 * 3];
    int higth, width;
    int index_catalog; // 下标
    int subscript;     // 数量
    catalogue()
    {
        index_catalog = 0;
        subscript = 0;
        fb = popen("ls /bmp", "r");
        if (fb == NULL)
        {
            cout << "popen error" << endl;
            exit(0);
        }
        else
        {
            char name[90] = {0};
            while (fgets(name, 90, fb) != NULL)
            {
                memset(catalogSet[subscript], '\0', 100);
                name[strlen(name) - 1] = '\0';
                sprintf(catalogSet[subscript++], "%s%s", PATh, name);
                memset(name, 0, 90);
                cout << catalogSet[subscript - 1] << endl;
            }
        }
    }

    // 读取图片像素
    void read_bmp(const char *filename)
    {
        int fp;
        fp = open(filename, O_RDWR);
        // cout << "fp=" << fp << endl;
        if (fp < 0)
        {
            perror("read open error");
            exit(0);
        }
        else
        {
            // 读取文件头
            lseek(fp, 18, SEEK_SET);
            read(fp, &width, 4);
            read(fp, &higth, 4);

            // 读取位图信息头
            memset(bmp_color, 0, sizeof(bmp_color));
            lseek(fp, 54, SEEK_SET);
            read(fp, bmp_color, width * higth * 3);
            // 关闭
            close(fp);
        }
    }
    ~catalogue()
    {
        pclose(fb);
    }

private:
    FILE *fb;
};

class lcd
{
public:
    lcd()
    {

        this->fd = open("/dev/fb0", O_RDWR);
        if (fd < 0)
        {
            cout << "lcd open error" << endl;
            exit(0);
        }
        else
        {
            cout << "lcd open success" << endl;
        }
        lcd_map = (int *)mmap(NULL, 800 * 480 * 4, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
        if (lcd_map == MAP_FAILED)
        {
            cout << "mmap error" << endl;
            exit(0);
        }
    }

    void show_bmp(char *bmp_color, int &width, int &higth)
    {
        for (int h = 0; h < 480; h++)
        {
            for (int w = 0; w < 800; w++)
            {
                int index = ((higth - 1 - h) * width + w) * 3;
                lcd_map[h * 800 + w] = (bmp_color[index]) | (bmp_color[index + 1] << 8) | (bmp_color[index + 2] << 16);
            }
        }
        cout << "show bmp success" << endl;
    }
    ~lcd()
    {
        munmap(lcd_map, 800 * 480 * 4);
        close(fd);
    }

private:
    int *lcd_map;
    int fd;
};

class touchScreen
{
private:
    int fd;

    // 按下前坐标
    int width, higth;

    // 松开坐标
    int width_end, higth_end;

public:
    // 滑动方向
    char slidingDirection[10];

    touchScreen()
    {

        fd = open("/dev/input/event0", O_RDWR);
        if (fd < 0)
        {
            cout << "touchScreen open error" << endl;
            exit(0);
        }
        else
        {
            cout << "touchScreen open success" << endl;
        }
    }

    // 计算坐标
    char *calculate_coordinate()
    {
        int flag = 0;
        int flag_end = 0;
        int temp_x = 0, temp_y = 0;
        cout << "calculate coordinate" << endl;
        while (1)
        {
            // 读取触摸屏事件信息
            struct input_event ev;
            read(fd, &ev, sizeof(struct input_event));
            if (ev.type == EV_ABS)
            {
                if (ev.code == ABS_X)
                {
                    temp_x = ev.value * 800 / 1024;
                }
                else if (ev.code == ABS_Y)
                {
                    temp_y = ev.value * 480 / 600;
                }
            }
            // 获取松开时的坐标
            if (temp_x != 0 && temp_y != 0)
            {

                if (ev.type == EV_KEY && ev.code == BTN_TOUCH && ev.value == 0 && flag_end == 0)
                {
                    width_end = temp_x;
                    higth_end = temp_y;
                    temp_x = 0;
                    temp_y = 0;
                    flag_end = 1;
                    cout << "width_end:" << width_end << "  higth_end:" << higth_end << endl;
                }

                // 获取按下时的坐标
                if (ev.type == EV_KEY && ev.code == BTN_TOUCH && ev.value == 1 && flag == 0)
                {
                    width = temp_x;
                    higth = temp_y;
                    temp_x = 0;
                    temp_y = 0;
                    flag = 1;
                    cout << "width:" << width << "  higth:" << higth << endl;
                }
            }
            if (width != 0 && higth != 0 && width_end != 0 && higth_end != 0)
            {
                break;
            }
        }
        if (width_end - width > 100)
        {
            strcpy(slidingDirection, "up");
            cout << "slidingDirection==" << slidingDirection << endl;
        }
        if (width_end - width < -100)
        {
            strcpy(slidingDirection, "down");
            cout << "slidingDirection==" << slidingDirection << endl;
        }
        if (width_end - width > 600 || width_end - width < -600)
        {
            strcpy(slidingDirection, "exit");
        }

        width_end = 0;
        higth_end = 0;
        width = 0;
        higth = 0;
        return slidingDirection;
    }
    void show_coordinate()
    {
        cout << "width:" << width << "  higth:" << higth << endl;
        cout << "width_end:" << width_end << "  higth_end:" << higth_end << endl;
    }

    ~touchScreen()
    {
        close(fd);
    }
};

void demo()
{
    lcd lcd;
    catalogue catalogue;
    catalogue.read_bmp(catalogue.catalogSet[catalogue.index_catalog]);
    lcd.show_bmp(catalogue.bmp_color, catalogue.width, catalogue.higth);
    touchScreen touch;
    while (1)
    {
        touch.calculate_coordinate();
        if (strcmp(touch.slidingDirection, "up") == 0)
        {
            catalogue.index_catalog--;
            if (catalogue.index_catalog < 0)
            {
                catalogue.index_catalog = catalogue.subscript - 1;
            }
            catalogue.read_bmp(catalogue.catalogSet[catalogue.index_catalog]);
            lcd.show_bmp(catalogue.bmp_color, catalogue.width, catalogue.higth);
        }
        else if (strcmp(touch.slidingDirection, "down") == 0)
        {
            catalogue.index_catalog++;
            if (catalogue.index_catalog > catalogue.subscript - 1)
            {
                catalogue.index_catalog = 0;
            }
            catalogue.read_bmp(catalogue.catalogSet[catalogue.index_catalog]);
            lcd.show_bmp(catalogue.bmp_color, catalogue.width, catalogue.higth);
        }
        else if (strcmp(touch.slidingDirection, "exit") == 0)
        {
            break;
        }
    }
}
int main(int argc, char const *argv[])
{
    demo();
    return 0;
}

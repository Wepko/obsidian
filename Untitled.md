## Практическая работа 3
**Вариант 4** 
4. Составить алгоритм решения задачи. Ввести предложение длиной не более 80 символов. Определить длину его последнего слова и количество слов короче последнего. Вывести эти слова. Количество пробелов между словами произвольно

~~~c++

[[include]] <iostream>

using namespace std;

string get_last_word(const string& str)
{
    if (str.length() == 0)
    {
        cerr << "Не строка\n";
        return 0;
    }

    int len = str.length();
    int i = len - 1;
    while (i >= 0 && str[i] != ' ')
    {
        i--;
    }
    string last_word;
    for (int j = i + 1; j < len; j++)
    {
        last_word += str[j];
    }
    return last_word;
}

int get_count_short_words(const string& str, int count_short_word) {
    
    int len = str.length();
    int count = 0;
    int count_char = 0;
    for (int i = 0; i < len; i++) {
        if (str[i] == ' ' && count_char > count_short_word) {
            count_char = 0;
            count++;
        }
        count_char++;
    }
    
    return  count;
}


int main()
{
    string str;

    cout << "Введите предложение \n";
    getline(cin, str);
    
    int last_word_ln = get_last_word(str).length();
    cout << "Длинна последнего слова: \n";
    cout << last_word_ln << endl;
    cout << "Количество слов короче последнего: \n";
    cout << get_count_short_words(str, last_word_ln) ;
    cout << endl;
}
~~~
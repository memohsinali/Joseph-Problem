# Joseph-Problem
#include<SFML\Graphics.hpp>
#include<SFML/Audio.hpp>
#include<iostream>
#include<Windows.h>
#include<string>
using namespace std;
using namespace sf;


template<class T>
class circularqueue
{

	class node
	{
	public:
		node* next;
		int id;
	};
	node* front;
	node* rear;
	int size;
public:


	//constructor
	circularqueue()
	{
		front = NULL;
		rear = NULL;
		size = 0;
	}

	class iterator
	{
		friend class circularqueue;

		node* curr;
	public:
		iterator(node* n = NULL)
		{
			curr = n;
		}
		iterator(iterator& obj)
		{
			curr = obj.curr;
		}

		iterator& operator++()
		{
			curr = curr->next;
			return *this;
		}

		int& operator*()
		{
			return curr->id;
		}

		bool operator !=(const iterator& itr) const
		{
			if (curr != itr.curr)
			{
				return true;
			}
			else
			{
				return false;
			}
		}


	};

	iterator begin()
	{
		iterator itr(front);
		return itr;
	}

	iterator end()
	{
		iterator itr(rear);
		return itr;
	}

	//enqueu function 

	void enqueue(int val)
	{
		if (front == NULL)
		{
			node* insert = new node;
			insert->id = val;
			front = insert;
			rear = insert;
			front->next = rear;
			rear->next = front;
			size++;
		}
		else
		{
			node* insert = new node;
			insert->id = val;
			rear->next = insert;
			rear = insert;
			rear->next = front;
			size++;
		}
	}

	void dequeue()
	{
		node* temp = front;
		front = front->next;
		rear->next = front;
		delete temp;
		size--;
	}

	//print function

	void print()
	{
		cout << "Player Id " << getRear()<< "    eleminated\n";
		iterator itr;
		itr = begin();
		cout << *itr << endl;
		++itr;
		for (;itr != begin();++itr)
		{
			cout << *itr << endl;
		}
	}

	int getSize()
	{
		return size;
	}

	T getFront()
	{
		return front->id;
	}

	T getRear()
	{
		return rear->id;
	}

	void post()
	{
		front = front->next;
		rear = rear->next;
	}

	~circularqueue()
	{
		/*if (front != nullptr)
		{
			node* temp1 = front;
			node* temp2 = front;
			while (temp1 != nullptr) 
		    {
				temp2 = temp2->next;
				delete temp1;
				temp1 = temp2;
		    }
	 	}*/
	}

};

void josephus(circularqueue<int> &q, int players, int start, int jumps)
{
	RenderWindow window(VideoMode(1920, 1080), "Sfml", Style::Default);
	

	Texture texture;                                     
	if (!texture.loadFromFile("soldier.jpg"))
	{
		cout << "Error loading character\n";
	}
	

	Texture cross;
	if (!cross.loadFromFile("cross.jpg"))                      
	{
		cout << "Error loading death character\n";
	}
	Font font;
	font.loadFromFile("text.ttf");

	Font sent;
	sent.loadFromFile("text.ttf");

	SoundBuffer buffer;                                    
	if (!buffer.loadFromFile("s2.wav"))
	{
		cout << "Error loading sound\n";
	}
	Sound sound;
	sound.setBuffer(buffer);

	Text sentence;

	Text* text = new Text[players];
	Sprite* soldier = new Sprite[players];
	circularqueue<int> ::iterator store;
	store = q.begin();
	CircleShape circle;
	circle.setRadius(300);
	circle.setFillColor(Color::Black);
	circle.setPosition(550, 45);
	circle.setPointCount(players);

	for (int i = 0; i < players; i++)
	{
		text[i].setFont(font);
		text[i].setString(to_string(i + 1));
		text[i].setFillColor(Color::Blue);
	}

	for (int i = 0; i < players; i++)
	{
		soldier[i].setTexture(texture);
		soldier[i].setPosition(circle.getPoint(i).x + 450, circle.getPoint(i).y);
		text[i].setPosition(circle.getPoint(i).x + 510, circle.getPoint(i).y+50);
	}

	bool line = 0;
	sentence.setFont(sent);
	sentence.setString("Let's play :) \n JOSEPHUS\n GAME\n");
	sentence.setCharacterSize(200);
	sentence.setFillColor(Color::Red);
	sentence.setOutlineColor(Color::Yellow);
	sentence.setOutlineThickness(10);
		bool flag = 0;
		int count = 1;
		circularqueue<int>::iterator itr;
		itr = q.begin();
		while (count != start)
		{
			count++;
			q.post();    //will give the node from where actually the game begins
		}
		window.setKeyRepeatEnabled(0);
		while (window.isOpen())
		{
			Event event;
			while (window.pollEvent(event))
			{
				if (event.type == Event::Closed)
				{
					window.close();
				}
				if (event.type == Event::KeyPressed)
				{

					if (event.key.code == Keyboard::Enter && q.getSize() > 1)
					{
						flag = 1;
					}
					else
					{
						return;
					}
				}
			}

			if (q.getSize() != 1 && flag == 1)
			{
				sound.play();
				for (int i = 0; i < jumps; i++)
				{
					q.post();
				}
				bool flag2 = true;
				int x = q.getFront();

				q.dequeue();

				soldier[x-1].setTexture(cross);
				flag = 0;
			}

			if (line == 0)
			{
				window.draw(sentence);
				window.display();
				line = 1;
				sleep(seconds(1));
			}
			
			window.clear();
			if(line!=0)
			{
				window.draw(circle);
				for (int i = 0; i < players; i++)
				{
					window.draw(soldier[i]);
					window.draw(text[i]);
				}
				window.display();
			}
			
		}
}


void menu()
{
	cout << "~~~~~~~~~~Hello Welcome to the Josephus Game~~~~~~~~~~\n";

	int players;
	int jumps;
	int start;
	cout << "Enter the total participants of game\n: ";
	cin >> players;
	cout << "Enter the number of jumps to be taken during game\n: ";
	cin >> jumps;
	cout << "Enter the id of player from where the game begins\n";
	cin >> start;
	circularqueue<int> q;
	for (int i = 0; i < players; i++)
	{
		q.enqueue(i + 1);
	}

	josephus(q,players,start,jumps-1);

}

int main()
{
	menu();
	return 0;
}

//David Snow
//COSC 3360
//Process Scheduling Assignment
//February 19, 2020

#include <iostream>
#include <queue>
#include <fstream>
#include <string>
#include <sstream>

using namespace std;

//FUNCTION HEADERS
void printArrival(int id);
void printNumCores(int id);
void printStart(int id);
void printRequestedCore(int id, int reqTime);
void printRequestedSSD(int id, int size);
void printRequestedPID(int id);
void printRequestedIO(int id, int reqTime);
void printWaitCore(int id, int size);
void printWaitSSD(int id, int size);
void printWaitIO(int id, int size);
void printCompletionCore(int id);
void printCompletionSSD(int id);
void printCompletionIO(int id);
void onCore(int id, int reqTime);
void onSSD(int id, int reqTime);
void onIO(int id, int reqTime);
void onCompleteCore(int id);
void onCompleteSSD(int id);
void Solution();
void printStartIO(int id);

//Global Variables (VAR hereafter)

int currTime = 0;
string NumCores = "NumCores";
string READY = "READY";
string START = "START";
string IDLE = "IDLE";
string CORE = "CORE";
string SSD = "SSD";
string IO = "IO";
string ARRIVAL = "ARRIVAL";
string TERMINATED = "TERMINATED";
string RUNNING = "RUNNING";
string BLOCKED = "BLOCKED";

//STRUCTS

struct request {
	string operation = "";
	int reqTime = 0;
};

struct process {
	int id = 0;
	int startTime = 0;
	int completionTime = 0;
	int coreAccessCount = 0;
	int coreRequestCount = 0;
	int SSDRequestCount = 0;
	int ssdAccessCount = 0;
	int ttyCount = 0;

	string status = IDLE;
	queue <request*> reqs; //core time request
};

void onProcessTerminated(process p, queue < process*> output, string core, string ssd);

void printNumCores(int id) {
	cout << "--NumCores event for process " << id << " at time " << currTime << " ms " << endl << endl;
}
void printStart(int id) {
	cout << "--Process" << id << " starts input from user at time " << currTime << " ms" << endl << endl;
}
void printPID(int id) {
	cout << "--Process " << id << " start input from user at time " << currTime << " ms" << endl << endl;
}
void printRequestCore(int id, int reqTime) {
	cout << "--Process" << id << "request a core at time " << currTime << " ms for" << reqTime << " ms" << endl << endl;
}

void printRequestSSD(int id, int reqTime) {
	cout << "--Process " << id << "request input from user at time " << currTime << " ms for" << reqTime << " ms" << endl << endl;
}
void printWaitCore(int id, int size) {
	cout << "--Process " << id << "must wait for a Core" << endl << endl;
	cout << "--Ready queue now contains " << size << " process(es) waiting for a CORE" << endl << endl;
}
void printWaitSSD(int id, int size) {
	cout << "--Process " << id << "must wait for a SSD\n\n" << "--Ready queue now contains " << size << " process(es) waiting for a SSD\n\n";
}
void printCompletionCore(int id) {
	cout << "--CORE completion event for a processs" << id << "at time " << currTime << " ms " << endl << endl;
}
void onCore(int id, int reqTime) {
	cout << "--Process " << id << " will complete core at time " << reqTime + currTime << " ms" << endl << endl;
}
void onSSD(int id, int reqTime) {
	cout << "--Process " << id << " will complete SSD at time " << reqTime + currTime << " ms" << endl << endl;
}
void onCompleteCore(int id) {
	cout << "CORE Completion event for process " << id << "at time " << currTime << " ms" << endl << endl;
}
void onCompleteSSD(int id) {
	cout << "SSD Completion event for process " << id << "at time " << currTime << " ms" << endl << endl;
}
//end of while loop
void onProcessTerminated(process p, queue<process*> output, string CORE, string SSD) {
	cout << "PROCESS TABLE\n\n";
	cout << "Process " << p.id << " terminates at time " << currTime << " ms\n\n";
	cout << "Process " << p.id << " ran for " << currTime - p.startTime << " ms, performed " << p.coreRequestCount << " and performed SSD Accesses of" << p.SSDRequestCount << " for " << SSD << " ms \n\n";
	cout << "Core is " << CORE << endl;
	cout << "SSD is " << SSD << endl;
}

int main() {
	Solution();
}

void print_queue(queue<process> q) {
	while (!q.empty()) {
		std::cout << q.front().id << "---" << endl << endl;
		std::cout << q.front().reqs.back()->reqTime << endl << endl;
		q.pop();
	}
}
void Solution() {
	std::ifstream infile("input.txt");
	string line;
	queue <process> processes;
	bool endline{};

	int id = 0;

	std::vector<int> v;
	process p;
	queue <process*> output;
	while (std::getline(infile, line))
	{
		std::stringstream iss(line);
		string a;
		int b;
		if (line != "") {
			if (!(iss >> a >> b)) {
				break;
			}
			request* r = new request;
			//cout << a << "" << b << endl;
			if (a == "NumCores") {
				if (p.status == ARRIVAL) {
					processes.push(p);
					process newp;
					p = newp;
				}
				p.startTime = b;
				p.id = id;
				v.push_back(id);
				id++;
				p.status = ARRIVAL;
			}
			else if (b != 0)
			{

				if (a == CORE) {
					r->operation = CORE;
					r->reqTime = b;
				}
				else if (a == CORE) {
					r->operation = CORE;
					r->reqTime = b;
				}
				else {
					std::cout << "File contains errors";
				}
				p.reqs.push(r);
			}

			if (a == SSD) {
				r->operation = SSD;
				r->reqTime = b;
			}
			else {
				std::cout << "File contains errors";
			}
			p.reqs.push(r);
		}
		else {
			if (a == START) {
				p.ssdAccessCount++;
			}
			int core_used_by = -1;
			queue<process> waitingQueue;
			queue<process> coreQueue;
			queue<process> ssdQueue;
			int SSD_used_by = -1;

			int processesSizetemp = processes.size();
			for (int i = 0; i < processesSizetemp; i++) {
				output.push(&processes.front());
			}

			string operation = "";
			process p;
			while (processes.size() > 0)
			{
				bool endline = false;
				currTime++;
				int processesSize = processes.size();
				for (int i = 0; i < processesSize; i++)
				{
					p = processes.front();
					p.completionTime++;
					processes.pop();
					if (p.status == ARRIVAL && p.startTime == currTime)
					{
						printArrival(p.id);
						printStart(p.id);
						p.status = READY;
					}
					if (p.status == CORE)
					{
						p.reqs.front()->reqTime--;
						if (p.reqs.front()->reqTime == 0)
						{
							core_used_by = -1;
							p.reqs.pop();
							printCompletionCore(p.id);
							p.status = READY;
							if (!waitingQueue.empty())
							{
								process temp = waitingQueue.front();
								waitingQueue.pop();
								core_used_by = temp.id;
								onCore(temp.id, temp.reqs.front()->reqTime);
								temp.status = CORE;
								processes.push(temp);
							}
						}
						if (p.status == SSD) {
							p.reqs.front()->reqTime--;
							if (p.reqs.front()->reqTime == 0) {
								SSD_used_by = -1;
								p.reqs.pop();
								printCompletionSSD(p.id);
								p.status = READY;
								if (!ssdQueue.empty()) {
									process temp = ssdQueue.front();
									ssdQueue.pop();
									SSD_used_by = temp.id;
									onSSD(temp.id, temp.reqs.front()->reqTime);
									temp.status = SSD;
									processes.push(temp);
								}
							}
						}
						if (p.reqs.empty()) {
							operation = p.reqs.front()->operation;

							if (operation == CORE && p.status == READY) {
								printRequestedCore(p.id, p.reqs.front()->reqTime);
								onCore(p.id, p.reqs.front()->reqTime);
								core_used_by = p.id;
								p.status = CORE;
							}
							else {
								p.status = BLOCKED;
								waitingQueue.push(p);
								printWaitCore(p.id, waitingQueue.size());
							}
							endline = true;
						}
						waitingQueue.push(p);
						printWaitCore(p.id, waitingQueue.size());
					}
					endline = true;
				}

				if (operation == SSD && p.status == READY) {
					printRequestedSSD(p.id, p.reqs.front()->reqTime);
					if (SSD_used_by == -1) {
						onSSD(p.id, p.reqs.front()->reqTime);
						p.ssdAccessCount++;
						SSD_used_by = p.id;
						p.status = SSD;
					}
					else {
						p.status = BLOCKED;
						ssdQueue.push(p);
						printWaitSSD(p.id, ssdQueue.size());
					}

					endline = true;
				}

				if (operation == IO && p.status == READY) {
					printRequestedIO(p.id, p.reqs.front()->reqTime);
					printStartIO(p.id);
					onIO(p.id, p.reqs.front()->reqTime);
					p.status = IO;
					endline = true;
				}
			}
			if (p.reqs.empty())
			{
				p.status = TERMINATED;
				onProcessTerminated(p, output, waitingQueue.empty() ? "IDLE" : "BUSY", ssdQueue.empty() ? "IDLE" : "BUSY");
				queue<process> dummy;
				dummy.push(p);
				queue<process> temp = processes;
				while (!temp.empty())
				{
					dummy.push(temp.front());
					temp.pop();
				}
				temp = ssdQueue;
				while (!temp.empty())
				{
					dummy.push(temp.front());
					temp.pop();
				}

				for (int m = 0; m < v.size(); m++) {
					queue <process> temp = dummy;
					while (!temp.empty()) {
						if (temp.front().id == v[m]) {
							process dtemp = temp.front();
							string status;
							if (dtemp.status == CORE) {
								status = RUNNING;
							}
							else if (dtemp.status == BLOCKED) {
								status = READY;
							}
							else if (dtemp.status == TERMINATED) {
								status = TERMINATED;
							}
							cout << "Process " << dtemp.id << " started at time " << dtemp.startTime << "ms and is " << status << endl;
							temp.pop();
						}
					}
				}
				if (p.status != TERMINATED && p.status != BLOCKED) {
					processes.push(p);
				}
			}
			else if(endline){
				cout << "\n";
			}
		}
	}
}

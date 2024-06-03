#include <iostream>
#include <vector>
#include <string>
#include <stdlib.h>
#include<fstream>

using namespace std;

template <typename T>

class Stack
{
private:
    vector<T> elements;

public:
    void push(const T& item)
    {
        elements.push_back(item);
    }

    void pop()
    {
        if (!elements.empty())
        {
            elements.pop_back();
        }
    }

    T top() const
    {
        if (!elements.empty())
        {
            return elements.back();
        }
        return T();  
    }

    bool empty() const
    {
        return elements.empty();
    }
};

class Course
{
public:
    string code;
    string name;
    vector<string> prerequisites;

    Course(string c, string n, vector<string> prereqs = {})
        : code(c), name(n), prerequisites(prereqs) {}
};

class Program
{
public:
    int numCourses;
    vector<Course> courses;

    Program(int n, const vector<Course>& c)
        : numCourses(n), courses(c) {}

    Program() : numCourses(0) {}
};

class Graph
{
private:
    vector<pair<string, vector<string>>> adjList;

public:
    void addEdge(const string& u, const string& v)
    {
        for (auto& pair : adjList)
        {
            if (pair.first == u)
            {
                pair.second.push_back(v);
                return;
            }
        }
        adjList.push_back({ u, {v} });
    }

    vector<string> getAdjList(const string& node) const
    {
        for (const auto& pair : adjList)
        {
            if (pair.first == node)
            {
                return pair.second;
            }
        }
        return {};
    }

    void displayGraph() const
    {
        for (const auto& pair : adjList)
        {
            cout << pair.first << ": ";
            for (const auto& v : pair.second)
            {
                cout << v << " ";
            }
            cout << endl;
        }
    }
};

class CoursePlanner
{
private:
    vector<Course> courses;
    Graph graph;

    bool isVisited(const vector<string>& visited, const string& node) const
    {
        for (const auto& v : visited)
        {
            if (v == node) return true;
        }
        return false;
    }

    void topologicalSortUtil(const string& v, vector<string>& visited, Stack<string>& stack)
    {
        visited.push_back(v);
        for (const auto& i : graph.getAdjList(v))
        {
            if (!isVisited(visited, i))
            {
                topologicalSortUtil(i, visited, stack);
            }
        }
        stack.push(v);
    }

    void findPathUtil(const string& course, vector<string>& path, vector<string>& visited) const 
    {
        if (isVisited(visited, course)) return;
        visited.push_back(course);
        for (const auto& prereq : getPrerequisites(course))
        {
            findPathUtil(prereq, path, visited);
        }
        path.push_back(course);
    }


    vector<string> getPrerequisites(const string& code) const
    {
        for (const auto& course : courses)
        {
            if (course.code == code)
            {
                return course.prerequisites;
            }
        }
        return {};
    }

public:
    void addCourse(const string& code, const string& name, const vector<string>& prereqs)
    {
        courses.emplace_back(code, name, prereqs);
        for (const auto& prereq : prereqs)
        {
            graph.addEdge(prereq, code);
        }
    }

    void displayCoursePlan(const vector<string>& desiredCourses) const 
    {
        for (const auto& course : desiredCourses)
        {
            vector<string> path;
            vector<string> visited;
            findPathUtil(course, path, visited);
            cout << "\t\t\t\t\t\t\t\tTo complete " << course << " (" << getCourseName(course) << "), you need to complete: ";
            for (const auto& p : path)
            {
                cout << p << " ";
            }
            cout << endl;
        }
    }


    string getCourseName(const string& code) const
    {
        for (const auto& course : courses)
        {
            if (course.code == code)
            {
                return course.name;
            }
        }
        return "";
    }

    void displayCourses() const
    {
        cout << "\t\t\t\t\t\t\t\tAvailable Courses:" << endl;
        for (const auto& course : courses)
        {
            cout << course.code << ": " << course.name << endl;
        }
    }

    void displayGraph() const
    {
        graph.displayGraph();
    }

    void topologicalSort()
    {
        vector<string> visited;
        Stack<string> stack;
        for (const auto& course : courses)
        {
            if (!isVisited(visited, course.code))
            {
                topologicalSortUtil(course.code, visited, stack);
            }
        }
        cout << "\t\t\t\t\t\t\t\tTopological Sort Order: ";
        while (!stack.empty())
        {
            cout << stack.top() << " ";
            stack.pop();
        }
        cout << endl;
    }
};

class Student
{
public:
    string name;
    double gpa;
    Program desiredProgram;
    vector<Course> completedCourses;

    Student(const string& n, double g, const Program& p)
        : name(n), gpa(g), desiredProgram(p) {}

    Student() : name(""), gpa(0.0) {}

    void displayProfile() const
    {
        cout << "\t\t\t\t\t\t\t\tStudent Profile" << endl;
        cout << "\t\t\t\t\t\t\t\tName: " << name << endl;
        cout << "\t\t\t\t\t\t\t\tGPA: " << gpa << endl;
        cout << "\t\t\t\t\t\t\t\tDesired Program Courses: ";
        for (const auto& course : desiredProgram.courses)
        {
            cout << course.code << " ";
        }
        cout << endl;
        cout << "\t\t\t\t\t\t\t\tCompleted Courses: ";
        for (const auto& course : completedCourses)
        {
            cout << course.code << " ";
        }
        cout << endl;
    }
    void saveProfile(const string& filename) const
    {
        ofstream outFile(filename);
        if (!outFile)
        {
            cout << "\t\t\t\t\t\t\t\tError opening file for writing." << endl;
            return;
        }
        outFile << name << endl;
        outFile << gpa << endl;
        outFile << desiredProgram.numCourses << endl;
        for (const auto& course : desiredProgram.courses)
        {
            outFile << course.code << " " << course.name << endl;
        }
        outFile << completedCourses.size() << endl;
        for (const auto& course : completedCourses)
        {
            outFile << course.code << " " << course.name << endl;
        }
        outFile.close();
    }

    void loadProfile(const string& filename)
    {
        ifstream inFile(filename);
        if (!inFile)
        {
            cout << "\t\t\t\t\t\t\t\tError opening file for reading." << endl;
            return;
        }
        getline(inFile, name);
        inFile >> gpa;
        int numCourses;
        inFile >> numCourses;
        inFile.ignore();
        vector<Course> programCourses;
        for (int i = 0; i < numCourses; ++i)
        {
            string code, name;
            getline(inFile, code, ' ');
            getline(inFile, name);
            programCourses.emplace_back(code, name);
        }
        desiredProgram = Program(numCourses, programCourses);

        int numCompletedCourses;
        inFile >> numCompletedCourses;
        inFile.ignore();
        completedCourses.clear();
        for (int i = 0; i < numCompletedCourses; ++i)
        {
            string code, name;
            getline(inFile, code, ' ');
            getline(inFile, name);
            completedCourses.emplace_back(code, name);
        }
        inFile.close();
    }
};

void displayPrograms(const vector<Program>& programs)
{
    cout << "\t\t\t\t\t\t\t\tPrograms:" << endl;

    for (const auto& program : programs)
    {
        if (program.numCourses == 8)
        {
            cout << endl <<"\t\t\t\t\t\t\t\tCOMPUTER SCIENCES" << endl;
        }
        else
        {
            cout << endl<< "\t\t\t\t\t\t\t\tMATHEMATICS" << endl;

        }
        cout << "\t\t\t\t\t\t\t\tProgram with " << program.numCourses << " courses:" << endl;
        {
            for (const auto& course : program.courses)
            {
                cout << course.code << ": " << course.name << endl;
            }
        }
    }
}

void displayRoadmap(const Program& program, const CoursePlanner& planner)
{
    vector<string> courseCodes;
    for (const auto& course : program.courses)
    {
        courseCodes.push_back(course.code);
    }
    planner.displayCoursePlan(courseCodes);
}

void updateStudentProfile(Student& student)
{
    cout << "\t\t\t\t\t\t\t\tUpdate Profile" << endl;
    cout << "\t\t\t\t\t\t\t\t1. Update Name" << endl;
    cout << "\t\t\t\t\t\t\t\t2. Update GPA" << endl;
    cout << "\t\t\t\t\t\t\t\t3. Update Completed Courses" << endl;
    cout << "\t\t\t\t\t\t\t\t4. Save Profile" << endl;   // New option
    cout << "\t\t\t\t\t\t\t\t5. Load Profile" << endl;   // New option
    cout << "\t\t\t\t\t\t\t\tEnter your choice: ";
    int choice;
    cin >> choice;
    cin.ignore();
    if (choice == 1)
    {
        cout << "\t\t\t\t\t\t\t\tEnter new name: ";
        getline(cin, student.name);
    }
    else if (choice == 2)
    {
        cout << "\t\t\t\t\t\t\t\tEnter new GPA: ";
        cin >> student.gpa;
    }
    else if (choice == 3)
    {
        int numCourses;
        cout << "\t\t\t\t\t\t\t\tEnter number of completed courses: ";
        cin >> numCourses;
        student.completedCourses.clear();
        for (int i = 0; i < numCourses; ++i)
        {
            string code, name;
            cout << "\t\t\t\t\t\t\t\tEnter course code and name: ";
            cin >> code;
            getline(cin, name);
            student.completedCourses.emplace_back(code, name);
        }
    }
    else if (choice == 4)
    {
        string filename;
        cout << "\t\t\t\t\t\t\t\tEnter filename to save profile: ";
        getline(cin, filename);
        student.saveProfile(filename);
    }
    else if (choice == 5)
    {
        string filename;
        cout << "\t\t\t\t\t\t\t\tEnter filename to load profile: ";
        getline(cin, filename);
        student.loadProfile(filename);
    }
    else
    {
        cout << "\t\t\t\t\t\t\t\tInvalid choice." << endl;
    }
}

Student createStudentProfile(const vector<Program>& programs)
{
    string name;
    double gpa;
    int programChoice;

    cout << "\t\t\t\t\t\t\t\tCreate Student Profile" << endl;
    cout << "\t\t\t\t\t\t\t\tEnter name: ";
    cin.ignore();
    getline(cin, name);
    cout << "\t\t\t\t\t\t\t\tEnter GPA: ";
    cin >> gpa;

    displayPrograms(programs);
    cout << "\t\t\t\t\t\t\t\tEnter the program number you want to enroll in: ";
    cin >> programChoice;

    if (programChoice < 1 || programChoice > programs.size())
    {
        cout << "\t\t\t\t\t\t\t\tInvalid program choice. Defaulting to an empty program." << endl;
        return Student(name, gpa, Program());
    }

    Program desiredProgram = programs[programChoice - 1];

    vector<Course> completedCourses;
    int numCompletedCourses;
    cout << "\t\t\t\t\t\t\t\tEnter number of completed courses: ";
    cin >> numCompletedCourses;
    cin.ignore(); 
    for (int i = 0; i < numCompletedCourses; ++i)
    {
        string code, name;
        cout << "\t\t\t\t\t\t\t\tEnter course code: ";
        getline(cin, code);
        cout << "\t\t\t\t\t\t\t\tEnter course name: ";
        getline(cin, name);
        completedCourses.emplace_back(code, name);
    }

    Student student(name, gpa, desiredProgram);
    student.completedCourses = completedCourses;
    return student;
}

Program inputProgram() {
    int numCourses;
    cout << "\t\t\t\t\t\t\t\tEnter the number of courses for the new program: ";
    cin >> numCourses;
    cin.ignore(); 

    vector<Course> courses;
    for (int i = 0; i < numCourses; ++i) {
        string code, name;
        cout << "\t\t\t\t\t\t\t\tEnter course code for course " << i + 1 << ": ";
        cin >> code;
        cin.ignore();
        cout << "\t\t\t\t\t\t\t\tEnter course name for course " << i + 1 << ": ";
        getline(cin, name);
        courses.emplace_back(code, name, vector<string>());
    }

    // Now, get prerequisites for each course
    for (auto& course : courses) {
        char hasPrerequisites;
        cout << "\t\t\t\t\t\t\t\tDoes course " << course.code << " have any prerequisites? (y/n): ";
        cin >> hasPrerequisites;
        cin.ignore(); 
        if (hasPrerequisites == 'y' || hasPrerequisites == 'Y') {
            int numPrerequisites;
            cout << "\t\t\t\t\t\t\t\tEnter the number of prerequisites for course " << course.code << ": ";
            cin >> numPrerequisites;
            cin.ignore(); //

            vector<string> prerequisites;
            for (int j = 0; j < numPrerequisites; ++j) {
                string prereq;
                cout << "\t\t\t\t\t\t\t\tEnter prerequisite " << j + 1 << " for course " << course.code << ": ";
                cin >> prereq;
                bool found = false;
                for (const auto& c : courses) {
                    if (c.code == prereq) {
                        found = true;
                        break;
                    }
                }
                if (!found) {
                    cout << "\t\t\t\t\t\t\t\tCourse " << prereq << " does not exist. Please enter a valid course code." << endl;
                    --j; // Decrement j to re-enter the prerequisite
                    continue;
                }
                prerequisites.push_back(prereq);
            }
            course.prerequisites = prerequisites; // Set prerequisites for the course
        }
    }

    return Program(numCourses, courses);
}




int main()
{
    vector<Program> programs;
    CoursePlanner planner;
    Student student;
    system("cls");
    // Adding programs and courses to the planner
    vector<Course> csCourses =
    {
        { "\t\t\t\t\t\t\t\tCS101", "Introduction to Computer Science", {} },
        { "\t\t\t\t\t\t\t\tCS102", "Data Structures", { "CS101" } },
        { "\t\t\t\t\t\t\t\tCS201", "Algorithms", { "CS102" } },
        { "\t\t\t\t\t\t\t\tCS202", "Advanced Algorithms", { "CS201" } },
        { "\t\t\t\t\t\t\t\tCS301", "Machine Learning", { "CS201", "CS202" } },
        { "\t\t\t\t\t\t\t\tCS302", "Deep Learning", { "CS301" } },
        { "\t\t\t\t\t\t\t\tCS401", "Artificial Intelligence", { "CS202" } },
        { "\t\t\t\t\t\t\t\tCS402", "Advanced AI", { "CS401" } }
    };

    vector<Course> mathCourses = {
        { "\t\t\t\t\t\t\t\tMATH101", "Calculus I", {} },
        { "\t\t\t\t\t\t\t\tMATH102", "Calculus II", { "MATH101" } },
        { "\t\t\t\t\t\t\t\tMATH201", "Linear Algebra", { "MATH102" } },
        { "\t\t\t\t\t\t\t\tMATH301", "Probability and Statistics", { "MATH201" } }
    };

    programs.push_back(Program(csCourses.size(), csCourses));
    programs.push_back(Program(mathCourses.size(), mathCourses));

    for (const auto& course : csCourses)
    {
        planner.addCourse(course.code, course.name, course.prerequisites);
    }

    for (const auto& course : mathCourses)
    {
        planner.addCourse(course.code, course.name, course.prerequisites);
    }

    while (true)
    {
        int choice;
        cout << endl << endl << endl << endl << endl;
        cout << "\n\t\t\t\t\t\t\t\tCOURSE PRE-REQUISITE PLANNER" << endl;
        cout << "\t\t\t\t\t\t\t\t1. Create Student Profile" << endl;
        cout << "\t\t\t\t\t\t\t\t2. Student Profile" << endl;
        cout << "\t\t\t\t\t\t\t\t3. List Programs and Courses" << endl;
        cout << "\t\t\t\t\t\t\t\t4. Program Roadmap" << endl;
        cout << "\t\t\t\t\t\t\t\t5. Update Student Profile" << endl;
        cout << "\t\t\t\t\t\t\t\t6. Add New Program and Courses" << endl; // New option
        cout << "\t\t\t\t\t\t\t\t7. Exit" << endl;
        cout << "\t\t\t\t\t\t\t\tEnter your choice: ";
        cin >> choice;

        if (choice == 1) {
            system("cls");
            cout << endl << endl << endl << endl << endl;
            student = createStudentProfile(programs);
        }
        else if (choice == 2) {
            system("cls");
            cout << endl << endl << endl << endl << endl;
            student.displayProfile();
        }
        else if (choice == 3) {
            system("cls");
            cout << endl << endl << endl << endl << endl;
            displayPrograms(programs);
        }
        else if (choice == 4) {
            system("cls");
            cout << endl << endl << endl << endl << endl;
            int programChoice;
            cout << "\t\t\t\t\t\t\t\tEnter the program number: ";
            cin >> programChoice;
            if (programChoice > 0 && programChoice <= programs.size()) {
                displayRoadmap(programs[programChoice - 1], planner);
            }
            else 
            {
                cout << endl << endl << endl << endl << endl;
                cout << "\t\t\t\t\t\t\t\tInvalid program number." << endl;
            }
        }
        else if (choice == 5) {
            system("cls");
            cout << endl << endl << endl << endl << endl;
            updateStudentProfile(student);
        }
        else if (choice == 6) { // Option to add new program and courses
            system("cls");
            cout << endl << endl << endl << endl << endl;
            programs.push_back(inputProgram());
        }
        else if (choice == 7) {
            break;
        }
        else {
            cout << endl << endl << endl << endl << endl;
            cout << "\t\t\t\t\t\t\t\tInvalid choice. Please try again." << endl;
        }
    }

    return 0;
}

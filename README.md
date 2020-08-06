# AlexProject

# import relevant modules
import statistics as stat
import matplotlib.pyplot as plot
import os

# define option 1
    # reads in grades from CSV file
    # stores grades in a multi-level dictionary for future use
    # closes file
    # computes average and letter grade for future options
    # if the csv was not structured correctly, will ask for a new CSV file path
def option_1():
    '''This function requests a file path, accesses the CSV file at the given location,
    stores the file's data in a dictionary for future access, and computes some summary figures.'''

    # create global gradebook that is available outside of the function
    path = input('\nProvide file path for Grades CSV: ')

    # reads in data. Allows path with or without file type extension.
    complete = 'no'
    while complete != 'yes':
        try:
            if path[-4:] == '.csv':
                with open(f'{path}') as grades:
                    lines = grades.readlines()
                    lines = [line.strip() for line in lines]
            else:
                with open(f'{path}.csv') as grades:
                    lines = grades.readlines()
                    lines = [line.strip() for line in lines]

            # organizes data in globally available gradebook
            global gradebook
            '''The gradebook contains all students' grades.'''

            gradebook = {}
            for line in lines[1:]:
                line = line.split(',')
                gradebook[line[0]] = {'Lab': [float(score) for score in line[1:7]],
                                      'Quiz': [float(score) for score in line[7:13]],
                                      'Reading': [float(score) for score in line[13:19]],
                                      'Exam': [float(score) for score in line[19:22]],
                                      'Project': float(line[22])}

            # calculate final grade and letter grade as multiple other functions will need these figures
            for grades in gradebook.values():
                grades['Score'] = (0.25 * stat.mean(grades['Lab']) +
                                          0.10 * stat.mean(grades['Quiz']) +
                                          0.10 * stat.mean(grades['Reading']) +
                                          0.45 * stat.mean(grades['Exam']) +
                                          0.10 * grades['Project'])
                if grades['Score'] < 59.5:
                    grades['Letter'] = 'F'
                elif grades['Score'] < 69.5:
                    grades['Letter'] = 'D'
                elif grades['Score'] < 79.5:
                    grades['Letter'] = 'C'
                elif grades['Score'] < 89.5:
                    grades['Letter'] = 'B'
                else:
                    grades['Letter'] = 'A'
            complete = 'yes'
        except:
            print('\nThe given file is not in the correct format.')
            path = input('\nProvide another file path for Grades CSV: ')


# define option 2
    # requests student UIN
    # error checking raises error if
        # UIN is not only numbers
        # UIN is not 10 digits long
        # UIN not found in grade center
    # if the UIN is not correct, will ask for new UIN until correct
    # if no errors thrown, calculates statistics for students
    # statistics are printed those to a file stored in the student's folder
def option_2():
    '''This function requests a student identification number, calculates select summary statistics for the student,
    and prints the figures to a TXT file named by the given identification number.'''
    student = input('\nProvide student UIN: ')
    complete = 'no'
    while complete != 'yes':
        try:
            # verify that UIN is 10 characters, only digits, and in grade center
            if not(student.isnumeric()):
                raise ValueError('\nThis UIN is invalid. The UIN must only contain digits.')
            if len(student) != 10:
                raise ValueError('\nThis UIN is invalid. The UIN must be exactly 10 digits.')
            if student not in gradebook:
                raise ValueError('\nThis student is not in the grade center.')

            # if an error is not thrown, check to see if directory for student exists in working directory
            # if direction does not already exist, create the directory
            student_folder = os.getcwd() + '\\' + student
            if os.path.isdir(student_folder) == False:
                os.makedirs(student_folder)

            # calculate summary statistics
            exams_mean = round(stat.mean(gradebook[student]['Exam']), 1)
            labs_mean = round(stat.mean(gradebook[student]['Lab']), 1)
            quizzes_mean = round(stat.mean(gradebook[student]['Quiz']), 1)
            readings_mean = round(stat.mean(gradebook[student]['Reading']), 1)
            final_score = round(gradebook[student]['Score'], 1)
            letter_grade = gradebook[student]['Letter']

            # write report to file
            with open(f'{student_folder}\\{student}.txt', 'w+') as individual_report:
                individual_report.write(f'Exams mean: {exams_mean}'
                                        f'\nLabs mean: {labs_mean}'
                                        f'\nQuizzes mean: {quizzes_mean}'
                                        f'\nReading activities mean: {readings_mean}'
                                        f'\nScore: {final_score}%'
                                        f'\nLetter grade: {letter_grade}')

            # reaching the end of this block of code will change value of variable to stop while loop
            complete = 'yes'

        # if an error thrown during UIN validation, the reason will be printed and a new UIN will be requested.
        except ValueError as exception:
            print(exception)
            student = input('\nProvide a correct student UIN: ')


# define option 3
    # requests student UIN
    # error checking raises error if
        # UIN is not only numbers
        # UIN is not 10 digits long
        # UIN not found in grade center
    # if the UIN is not correct, will ask for new UIN until correct
    # if no errors thrown, plots the students grades
    # plots are saved as PNGs in a directory with UIN as name
def option_3():
    '''This function requests a student identification number and plots the given student's
    scores on Exams, Labs, Quizzes, and Readings, which are saved as PNG files in the student's folder.'''
    student = input('\nProvide student UIN: ')
    complete = 'no'
    while complete != 'yes':
        try:
            # verify that UIN is 10 characters, only digits, and in grade center
            if not (student.isnumeric()):
                raise ValueError('This UIN is invalid. The UIN must only contain digits.')
            if len(student) != 10:
                raise ValueError('This UIN is invalid. The UIN must be exactly 10 digits.')
            if student not in gradebook:
                raise ValueError('This student is not in the grade center.')

            # if an error is not thrown, check to see if directory for student exists in working directory
            # if direction does not already exist, create the directory
            student_folder = os.getcwd() + '\\' + student
            if os.path.isdir(student_folder) == False:
                os.makedirs(student_folder)

            # plot grades for each criteria/type of assignment: Exam, Lab, Quiz, Reading
            colors = ['deepskyblue', 'deeppink', 'mediumspringgreen', 'blueviolet', 'yellow','orange']
            criteria = ['Exam', 'Lab', 'Quiz', 'Reading']
            for criterion in criteria:
                grades = gradebook[student][criterion]
                labels = []
                for item in range(len(grades)):
                    labels.append(criterion + ' ' + str(item + 1))
                bars = plot.bar(labels, grades)
                plot.xlabel(f'{criterion} Number')
                plot.ylabel('Grade')
                plot.title(f'CSCE 110 {criterion} Grades')
                for n in range(len(bars)):
                    bars[n].set_color(colors[n])
                file_name = student_folder + '\\' + criterion + ' Grade Distribution.png'
                plot.savefig(file_name)
                plot.clf()
            complete = 'yes'

        # if an error thrown during UIN validation, the reason will be printed and a new UIN will be requested.
        except ValueError as exception:
            print(exception)
            student = input('\nProvide a correct student UIN: ')


# define option 4
    # pulls final scores out of the gradebook into seperate list
    # calculates summary statistics
    # opens a file to write a brief report
def option_4():
    '''This function calculates summary statistics on the final scores of the entire class
    and writes the statistics to a TXT file called "report".'''

    # extract scores and save in list named final_scores
    total_students = len(gradebook)
    final_scores = []
    for grades in gradebook.values():
        final_scores.append(grades['Score'])

    # calculate summary statistics
    minimum = round(min(final_scores),1)
    maximum = round(max(final_scores),1)
    median = round(stat.median(final_scores),1)
    average = round(stat.mean(final_scores),1)
    sigma = round(stat.pstdev(final_scores),1)

    # write summary statistics to txt file
    with open('report.txt', 'w+') as class_report:
        class_report.write(f'Total number of students: {total_students}'
                           f'\nMinimum score: {minimum}'
                           f'\nMaximum score: {maximum}'
                           f'\nMedian score: {median}'
                           f'\nMean score: {average}'
                           f'\nStandard deviation: {sigma}')


# define option 5
    # extracts letter grades from the gradebook and creates summary plots
    # plots are saved in folder class_charts
def option_5():
    '''This function plots the distribution of the class's letter grades in both a pie and a bar which are saved
    as PNG files in a folder called "class_charts".'''
    # extract letter grades and count frequency
    final_letters = []
    for grades in gradebook.values():
        final_letters.append(grades['Letter'])
    letter_grades = ['A', 'B', 'C', 'D', 'F']
    letter_counts = []
    for letter in letter_grades:
        letter_counts.append(final_letters.count(letter))

    # check to see if directory "class_charts" exists in working directory
    # if directory does not exist, create the directory
    class_folder = os.getcwd() + '\\' + 'class_charts'
    if os.path.isdir(class_folder) == False:
        os.makedirs(class_folder)

    # plot
    colors = ['deepskyblue', 'deeppink', 'mediumspringgreen', 'blueviolet', 'yellow', 'orange']

    # bar plot
    bars = plot.bar(letter_grades, letter_counts)
    plot.ylabel('Number of Students')
    plot.xlabel('Letter Grade')
    plot.title(f'CSCE 110 Letter Grade Distribution')
    for n in range(len(bars)):
        bars[n].set_color(colors[n])
    file_name = class_folder + '\\' + 'Grade Distribution Bar Chart.png'
    plot.savefig(file_name)
    plot.clf()

    # pie plot
    plot.title(f'CSCE 110 Letter Grade Distribution')
    plot.pie(letter_counts, labels=letter_grades, colors=colors)
    for n in range(len(bars)):
        bars[n].set_color(colors[n])
    file_name = class_folder + '\\' + 'Grade Distribution Pie Chart.png'
    plot.savefig(file_name)
    plot.clf()


# define main function
    # loops until q, quiz, or 6 is entered
    # allowing selection of different options
def main():
    '''This function requests an option to execute, executes the corresponding option function, then asks
    the user to select another option.'''

    # assign main menu
    main_menu = '\n******************* Main Menu *******************' \
                '\n1. Read CSV file of grades' \
                '\n2. Generate student report file' \
                '\n3. Generate student report charts' \
                '\n4. Generate class report file' \
                '\n5. Generate class report charts' \
                '\n6. Quit' \
                '\n*************************************************'

    # use dictionary to define which option will be called by inputs
    input_options = {'1': option_1,
                     '2': option_2,
                     '3': option_3,
                     '4': option_4,
                     '5': option_5}

    # request option '1' or '6' then request other options until "q" is entered
    data_read = 'no'
    print(main_menu)
    option = input('\nSelect press "1" to begin or press "q" to quit: ')

    while (option.lower() != 'q') and (option.lower() != 'quit') and (option != '6'):
        if data_read == 'no':
            if option == '1':
                data_read = 'yes'
                input_options[option]()
                print(f'\nOption {option} complete.')
                print(main_menu)
                option = input('\nSelect another option or press "q" to quit: ')
            else:
                print('\nInvalid selection. Option 1 must be selected before options 2 - 5.')
                print(main_menu)
                option = input('\nSelect press "1" to begin or press "q" to quit: ')
        else:
            if option in input_options:
                input_options[option]()
                print(f'\nOption {option} complete.')
                print(main_menu)
                option = input('\nSelect another option or press "q" to quit: ')
            else:
                print('\nInvalid selection.')
                print(main_menu)
                option = input('\nSelect an option 1 - 5 or press "q" to quit: ')
    print('\nSession Concluded.')

# call main function
main()

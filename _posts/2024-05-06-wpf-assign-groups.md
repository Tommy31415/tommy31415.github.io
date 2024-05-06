---
layout: post
title: Assign users to groups with WPF
date: 2024-05-06
categories: wpf
---

## UI initial design


```xml
<Window x:Class="YourNamespace.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:telerik="http://schemas.telerik.com/2008/xaml/presentation"
        Title="User Management" Height="450" Width="800">
    <Grid>
        <!-- Filter Controls -->
        <StackPanel Orientation="Horizontal" Margin="10">
            <TextBox Text="{Binding FilterText, UpdateSourceTrigger=PropertyChanged}" Width="200" Margin="0 0 10 0"/>
            <telerik:RadComboBox ItemsSource="{Binding Groups}" SelectedItem="{Binding SelectedGroup}" DisplayMemberPath="Name" Width="200"/>
        </StackPanel>

        <!-- User List -->
        <telerik:RadGridView ItemsSource="{Binding FilteredUsers}" SelectedItem="{Binding SelectedUser}" AutoGenerateColumns="False" Margin="10">
            <!-- Define Columns -->
        </telerik:RadGridView>

        <!-- Group Assignment -->
        <StackPanel Orientation="Horizontal" Margin="10">
            <telerik:RadListBox ItemsSource="{Binding Groups}" SelectedItems="{Binding SelectedGroups}" DisplayMemberPath="Name" Width="200"/>
            <Button Content="Assign Groups" Command="{Binding AssignGroupsCommand}" Margin="10 0"/>
        </StackPanel>
    </Grid>
</Window>
```

## Initial backend design

```c#
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Linq;
using System.Windows;
using System.Windows.Input;
using Telerik.Windows.Controls;

namespace YourNamespace
{
    public class MainViewModel : INotifyPropertyChanged
    {
        // List of all users
        private ObservableCollection<User> _users;
        public ObservableCollection<User> Users
        {
            get { return _users; }
            set
            {
                _users = value;
                OnPropertyChanged(nameof(Users));
            }
        }

        // Filtered list of users based on name and group filtering
        private ObservableCollection<User> _filteredUsers;
        public ObservableCollection<User> FilteredUsers
        {
            get { return _filteredUsers; }
            set
            {
                _filteredUsers = value;
                OnPropertyChanged(nameof(FilteredUsers));
            }
        }

        // Selected user
        private User _selectedUser;
        public User SelectedUser
        {
            get { return _selectedUser; }
            set
            {
                _selectedUser = value;
                OnPropertyChanged(nameof(SelectedUser));
            }
        }

        // List of all groups
        private ObservableCollection<Group> _groups;
        public ObservableCollection<Group> Groups
        {
            get { return _groups; }
            set
            {
                _groups = value;
                OnPropertyChanged(nameof(Groups));
            }
        }

        // Selected group
        private Group _selectedGroup;
        public Group SelectedGroup
        {
            get { return _selectedGroup; }
            set
            {
                _selectedGroup = value;
                OnPropertyChanged(nameof(SelectedGroup));
            }
        }

        // Selected groups for assignment
        private ObservableCollection<Group> _selectedGroups;
        public ObservableCollection<Group> SelectedGroups
        {
            get { return _selectedGroups; }
            set
            {
                _selectedGroups = value;
                OnPropertyChanged(nameof(SelectedGroups));
            }
        }

        // Filter text for name filtering
        private string _filterText;
        public string FilterText
        {
            get { return _filterText; }
            set
            {
                _filterText = value;
                OnPropertyChanged(nameof(FilterText));
                ApplyFilter();
            }
        }

        // Command to assign selected groups to the selected user
        public ICommand AssignGroupsCommand { get; set; }

        public MainViewModel()
        {
            // Initialize collections and commands
            Users = new ObservableCollection<User>(GetAllUsers());
            FilteredUsers = new ObservableCollection<User>(Users);
            Groups = new ObservableCollection<Group>(GetAllGroups());
            SelectedGroups = new ObservableCollection<Group>();

            AssignGroupsCommand = new RelayCommand(ExecuteAssignGroups, CanExecuteAssignGroups);
        }

        // Method to apply filtering based on filter text and selected group
        private void ApplyFilter()
        {
            FilteredUsers.Clear();
            var filtered = Users.Where(u => u.Name.Contains(FilterText));
            if (SelectedGroup != null)
            {
                filtered = filtered.Where(u => u.Groups.Contains(SelectedGroup));
            }
            foreach (var user in filtered)
            {
                FilteredUsers.Add(user);
            }
        }

        // Command execution method to assign selected groups to the selected user
        private void ExecuteAssignGroups(object parameter)
        {
            if (SelectedUser != null && SelectedGroups != null && SelectedGroups.Any())
            {
                foreach (var group in SelectedGroups)
                {
                    SelectedUser.Groups.Add(group);
                }
            }
        }

        // Command can execute method for assigning groups
        private bool CanExecuteAssignGroups(object parameter)
        {
            return SelectedUser != null && SelectedGroups != null && SelectedGroups.Any();
        }

        // Dummy method to get all users (replace with your actual data retrieval logic)
        private List<User> GetAllUsers()
        {
            return new List<User>
            {
                new User { Id = 1, Name = "User 1", Groups = new List<Group>() },
                new User { Id = 2, Name = "User 2", Groups = new List<Group>() },
                new User { Id = 3, Name = "User 3", Groups = new List<Group>() }
            };
        }

        // Dummy method to get all groups (replace with your actual data retrieval logic)
        private List<Group> GetAllGroups()
        {
            return new List<Group>
            {
                new Group { Id = 1, Name = "Group 1" },
                new Group { Id = 2, Name = "Group 2" },
                new Group { Id = 3, Name = "Group 3" }
            };
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged(string propertyName)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }

    // Model classes for User and Group
    public class User
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public List<Group> Groups { get; set; }
    }

    public class Group
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }

    // RelayCommand implementation for ICommand
    public class RelayCommand : ICommand
    {
        private readonly Action<object> _execute;
        private readonly Func<object, bool> _canExecute;

        public RelayCommand(Action<object> execute, Func<object, bool> canExecute = null)
        {
            _execute = execute;
            _canExecute = canExecute;
        }

        public event EventHandler CanExecuteChanged
        {
            add { CommandManager.RequerySuggested += value; }
            remove { CommandManager.RequerySuggested -= value; }
        }

        public bool CanExecute(object parameter)
        {
            return _canExecute == null || _canExecute(parameter);
        }

        public void Execute(object parameter)
        {
            _execute(parameter);
        }
    }
}

```
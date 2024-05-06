---
layout: post
title: Assign users to groups with WPF
date: 2024-05-06
categories: wpf
---

## UI initial design

```xaml
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
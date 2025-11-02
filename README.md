# repo-for-test 
#!/usr/bin/env python3
"""
 Manager CLI - A simple command-line task management tool
"""

import argparse
import sys
from tasks.manager import TaskManager
from tasks.models import Task, Priority

def main():
    parser = argparse.ArgumentParser(description='Task Manager CLI')
    subparsers = parser.add_subparsers(dest='command', help='Available commands')
    
    # Add task command
    add_parser = subparsers.add_parser('add', help='Add a new task')
    add_parser.add_argument('title', help='Task title')
    add_parser.add_argument('-d', '--description', help='Task description')
    add_parser.add_argument('-p', '--priority', choices=['low', 'medium', 'high'], 
                           default='medium', help='Task priority')
    
    # List tasks command
    list_parser = subparsers.add_parser('list', help='List all tasks')
    list_parser.add_argument('-s', '--status', choices=['pending', 'completed'], 
                            help='Filter by status')
    
    # Complete task command
    complete_parser = subparsers.add_parser('complete', help='Mark task as completed')
    complete_parser.add_argument('task_id', type=int, help='Task ID to complete')
    
    # Delete task command
    delete_parser = subparsers.add_parser('delete', help='Delete a task')
    delete_parser.add_argument('task_id', type=int, help='Task ID to delete')
    
    # Stats command
    subparsers.add_parser('stats', help='Show task statistics')
    
    args = parser.parse_args()
    
    if not args.command:
        parser.print_help()
        sys.exit(1)
    
    manager = TaskManager()
    
    try:
        if args.command == 'add':
            task = Task(
                title=args.title,
                description=args.description,
                priority=Priority(args.priority.upper())
            )
            manager.add_task(task)
            print(f"✅ Task added: {task.title}")
        
        elif args.command == 'list':
            tasks = manager.list_tasks(status=args.status)
            if not tasks:
                print("No tasks found.")
            else:
                for task in tasks:
                    status = "✓" if task.completed else "○"
                    print(f"[{task.id}] {status} {task.title} ({task.priority.value})")
                    if task.description:
                        print(f"    └─ {task.description}")
        
        elif args.command == 'complete':
            if manager.complete_task(args.task_id):
                print(f"✅ Task {args.task_id} marked as completed")
            else:
                print(f"❌ Task {args.task_id} not found")
        
        elif args.command == 'delete':
            if manager.delete_task(args.task_id):
                print(f"🗑️  Task {args.task_id} deleted")
            else:
                print(f"❌ Task {args.task_id} not found")
        
        elif args.command == 'stats':
            stats = manager.get_stats()
            print(f"📊 Task Statistics:")
            print(f"   Total: {stats['total']}")
            print(f"   Pending: {stats['pending']}")
            print(f"   Completed: {stats['completed']}")
    
    except Exception as e:
        print(f"❌ Error: {e}")
        sys.exit(1)

if __name__ == '__main__':
    main()

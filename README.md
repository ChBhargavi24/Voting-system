import json

candidates = []
votes = []
voted_ids = set()
admin_password = "admin123"
file_name = "voting_data.json"

def save_data():
with open(file_name, "w") as f:
json.dump(
{
"candidates": candidates,
"votes": votes,
"voted_ids": list(voted_ids),
},
f,
)

def load_data():
"""Load data safely even if older file lacks some keys."""
global candidates, votes, voted_ids
try:
with open(file_name, "r") as f:
data = json.load(f)
except (FileNotFoundError, json.JSONDecodeError):
data = {}

candidates[:] = data.get("candidates", [])  
votes_from_file = data.get("votes")  

if isinstance(votes_from_file, list):  
    votes[:] = votes_from_file  
else:  
    votes[:] = [0] * len(candidates)  

if len(votes) < len(candidates):  
    votes.extend([0] * (len(candidates) - len(votes)))  
elif len(votes) > len(candidates):  
    votes[:] = votes[: len(candidates)]  

voted_ids.clear()  
voted_ids.update(data.get("voted_ids", []))

def show_results():
if not candidates:
print("No candidates available.")
return

print("\n--- Voting Results ---")  
total_votes = sum(votes)  
for name, count in zip(candidates, votes):  
    print(f"{name}: {count} vote(s)")  

if total_votes == 0:  
    print("\nNo votes have been cast yet.")  
    return  

max_votes = max(votes)  
winners = [candidates[i] for i, v in enumerate(votes) if v == max_votes]  

if len(winners) == 1:  
    print(f"\nWinner: {winners[0]} with {max_votes} vote(s)")  
else:  
    print(f"\nIt's a tie between: {', '.join(winners)} with {max_votes} vote(s) each")

def reset_voting():
global candidates, votes, voted_ids
candidates.clear()
votes.clear()
voted_ids.clear()
save_data()
print("Voting system reset successfully. You can add new candidates now.")

def run_voting():
load_data()
while True:
print("\n--- Voting System ---")
print("1. Add Candidates (Admin Only)")
print("2. Show Candidates")
print("3. Cast Votes")
print("4. Show Results")
print("5. Reset Voting (Admin Only)")
print("6. Exit")

choice = input("Choose an option: ").strip()  

    if choice == "1":  
        pwd = input("Enter Admin Password: ").strip()  
        if pwd != admin_password:  
            print("Wrong password! Access denied.")  
            continue  

        try:  
            num_candidates = int(input("Enter the number of candidates: ").strip())  
        except ValueError:  
            print("Invalid input. Please enter a number.")  
            continue  

        for i in range(num_candidates):  
            name = input(f"Enter name of candidate {i+1}: ").strip()  
            if not name:  
                print("Empty name skipped.")  
                continue  
            if name.lower() in (c.lower() for c in candidates):  
                print(f"Candidate '{name}' already exists, skipping...")  
            else:  
                candidates.append(name)  

        votes[:] = [0] * len(candidates)  
        print("Candidates added successfully!")  
        save_data()  

    elif choice == "2":  
        if not candidates:  
            print("No candidates yet.")  
        else:  
            print("\n--- Candidates ---")  
            for i, c in enumerate(candidates, start=1):  
                print(f"{i}. {c}")  

    elif choice == "3":  
        if not candidates:  
            print("No candidates to vote for.")  
            continue  

        try:  
            num_voters = int(input("Enter number of voters: ").strip())  
        except ValueError:  
            print("Invalid input. Please enter a number.")  
            continue  

        for v in range(1, num_voters + 1):  
            print(f"\nVoter {v}, please enter your details:")  
            voter_id = input("Enter your Voter ID: ").strip()  

            if voter_id in voted_ids:  
                print("You have already voted!")  
                continue  

            print("\nCandidates:")  
            for i, c in enumerate(candidates, start=1):  
                print(f"{i}. {c}")  

            try:  
                choice_num = int(input("Enter candidate number to vote: ").strip())  
                if 1 <= choice_num <= len(candidates):  
                    votes[choice_num - 1] += 1  
                    voted_ids.add(voter_id)  
                    print("Vote cast successfully!")  
                else:  
                    print("Invalid candidate number, vote skipped.")  
            except ValueError:  
                print("Invalid input, vote skipped.")  

        save_data()  

    elif choice == "4":  
        show_results()  

    elif choice == "5":  
        pwd = input("Enter Admin Password: ").strip()  
        if pwd == admin_password:  
            reset_voting()  
        else:  
            print("Wrong password! Access denied.")  

    elif choice == "6":  
        save_data()  
        print("Voting Ended Successfully.......!")  
        break  

    else:  
        print("Invalid option, try again.")

candidates.clear()
votes.clear()
voted_ids.clear()
save_data()

run_voting()

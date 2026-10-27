import csv
import os

CSV_FILE = "candidates.csv"


class Candidate:
    def __init__(self, candidate_id, name, votes=0):
        self.candidate_id = candidate_id
        self.name = name
        self.votes = votes
        self.next = None


class VotingSystem:
    def __init__(self):
        self.head = None
        self.load_candidates()

    def find_candidate(self, candidate_id):
        current = self.head
        while current is not None:
            if current.candidate_id == candidate_id:
                return current
            current = current.next
        return None

    def load_candidates(self):
        if not os.path.exists(CSV_FILE):
            return

        with open(CSV_FILE, "r", newline="", encoding="utf-8") as file:
            reader = csv.DictReader(file)

            for row in reader:
                candidate = Candidate(
                    row["candidate_id"],
                    row["name"],
                    int(row["votes"])
                )

                if self.head is None:
                    self.head = candidate
                else:
                    current = self.head
                    while current.next is not None:
                        current = current.next
                    current.next = candidate

    def save_candidates(self):
        with open(CSV_FILE, "w", newline="", encoding="utf-8") as file:
            fieldnames = ["candidate_id", "name", "votes"]
            writer = csv.DictWriter(file, fieldnames=fieldnames)
            writer.writeheader()

            current = self.head
            while current is not None:
                writer.writerow({
                    "candidate_id": current.candidate_id,
                    "name": current.name,
                    "votes": current.votes
                })
                current = current.next

    def register_candidate(self, candidate_id, name):
        if self.find_candidate(candidate_id) is not None:
            print("Candidate ID already exists.")
            return

        new_candidate = Candidate(candidate_id, name)

        if self.head is None:
            self.head = new_candidate
        else:
            current = self.head
            while current.next is not None:
                current = current.next
            current.next = new_candidate

        self.save_candidates()
        print("Candidate registered successfully.")

    def cast_vote(self, candidate_id):
        candidate = self.find_candidate(candidate_id)
        if candidate is None:
            print("Candidate not found.")
            return

        candidate.votes += 1
        self.save_candidates()
        print("Vote cast successfully for", candidate.name)

    def count_votes(self):
        if self.head is None:
            print("No candidates registered.")
            return

        current = self.head
        print("\n--- Vote Count ---")

        while current is not None:
            print(
                "ID:", current.candidate_id,
                "| Name:", current.name,
                "| Votes:", current.votes
            )
            current = current.next

    def display_winner(self):
        if self.head is None:
            print("No candidates registered.")
            return

        highest_votes = self.head.votes
        current = self.head.next

        while current is not None:
            highest_votes = max(highest_votes, current.votes)
            current = current.next

        if highest_votes == 0:
            print("No votes have been cast yet.")
            return

        winners = []
        current = self.head

        while current is not None:
            if current.votes == highest_votes:
                winners.append(current)
            current = current.next

        if len(winners) == 1:
            print("Winner:", winners[0].name)
            print("Votes:", winners[0].votes)
        else:
            print("Election is tied between:")
            for candidate in winners:
                print(candidate.name, "-", candidate.votes, "votes")


def main():
    voting_system = VotingSystem()

    while True:
        print("\n--- Voting System ---")
        print("1. Register candidate")
        print("2. Cast vote")
        print("3. Show vote count")
        print("4. Show winner")
        print("5. Exit")

        choice = input("Enter your choice: ").strip()

        if choice == "1":
            candidate_id = input("Enter candidate ID: ").strip()
            name = input("Enter candidate name: ").strip()
            if candidate_id and name:
                voting_system.register_candidate(candidate_id, name)
            else:
                print("Candidate ID and name cannot be empty.")

        elif choice == "2":
            candidate_id = input("Enter candidate ID to vote for: ").strip()
            voting_system.cast_vote(candidate_id)

        elif choice == "3":
            voting_system.count_votes()

        elif choice == "4":
            voting_system.display_winner()

        elif choice == "5":
            print("Exiting voting system.")
            break

        else:
            print("Invalid choice. Please enter 1 to 5.")


if __name__ == "__main__":
    main()
